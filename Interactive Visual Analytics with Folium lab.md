<p style="text-align:center">
    <a href="https://skills.network/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo"  />
    </a>
</p>


# **Launch Sites Locations Analysis with Folium**


Estimated time needed: **40** minutes


The launch success rate may depend on many factors such as payload mass, orbit type, and so on. It may also depend on the location and proximities of a launch site, i.e., the initial position of rocket trajectories. Finding an optimal location for building a launch site certainly involves many factors and hopefully we could discover some of the factors by analyzing the existing launch site locations.


In the previous exploratory data analysis labs, you have visualized the SpaceX launch dataset using `matplotlib` and `seaborn` and discovered some preliminary correlations between the launch site and success rates. In this lab, you will be performing more interactive visual analytics using `Folium`.


## Objectives


This lab contains the following tasks:

*   **TASK 1:** Mark all launch sites on a map
*   **TASK 2:** Mark the success/failed launches for each site on the map
*   **TASK 3:** Calculate the distances between a launch site to its proximities

After completed the above tasks, you should be able to find some geographical patterns about launch sites.


Let's first import required Python packages for this lab:



```python
!pip3 install folium
!pip3 install wget
```

    Collecting folium
      Downloading folium-0.12.1.post1-py2.py3-none-any.whl (95 kB)
    [K     |████████████████████████████████| 95 kB 5.1 MB/s eta 0:00:011
    [?25hRequirement already satisfied: numpy in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from folium) (1.20.3)
    Requirement already satisfied: jinja2>=2.9 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from folium) (3.0.2)
    Requirement already satisfied: requests in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from folium) (2.26.0)
    Collecting branca>=0.3.0
      Downloading branca-0.5.0-py3-none-any.whl (24 kB)
    Requirement already satisfied: MarkupSafe>=2.0 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from jinja2>=2.9->folium) (2.0.1)
    Requirement already satisfied: urllib3<1.27,>=1.21.1 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from requests->folium) (1.26.7)
    Requirement already satisfied: certifi>=2017.4.17 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from requests->folium) (2022.6.15)
    Requirement already satisfied: idna<4,>=2.5 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from requests->folium) (3.3)
    Requirement already satisfied: charset-normalizer~=2.0.0 in /opt/conda/envs/Python-3.9/lib/python3.9/site-packages (from requests->folium) (2.0.4)
    Installing collected packages: branca, folium
    Successfully installed branca-0.5.0 folium-0.12.1.post1
    Collecting wget
      Downloading wget-3.2.zip (10 kB)
    Building wheels for collected packages: wget
      Building wheel for wget (setup.py) ... [?25ldone
    [?25h  Created wheel for wget: filename=wget-3.2-py3-none-any.whl size=9672 sha256=f652b314ce1bcb61dd75d2a0ade11636b57528334222787c1af116de08ccba62
      Stored in directory: /tmp/wsuser/.cache/pip/wheels/04/5f/3e/46cc37c5d698415694d83f607f833f83f0149e49b3af9d0f38
    Successfully built wget
    Installing collected packages: wget
    Successfully installed wget-3.2



```python
import folium
import wget
import pandas as pd
```


```python
# Import folium MarkerCluster plugin
from folium.plugins import MarkerCluster
# Import folium MousePosition plugin
from folium.plugins import MousePosition
# Import folium DivIcon plugin
from folium.features import DivIcon
```

If you need to refresh your memory about folium, you may download and refer to this previous folium lab:


[Generating Maps with Python](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module\_3/DV0101EN-3-5-1-Generating-Maps-in-Python-py-v2.0.ipynb)


## Task 1: Mark all launch sites on a map


First, let's try to add each site's location on a map using site's latitude and longitude coordinates


The following dataset with the name `spacex_launch_geo.csv` is an augmented dataset with latitude and longitude added for each site.



```python
# Download and read the `spacex_launch_geo.csv`
spacex_csv_file = wget.download('https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/datasets/spacex_launch_geo.csv')
spacex_df=pd.read_csv(spacex_csv_file)
```

Now, you can take a look at what are the coordinates for each site.



```python
# Select relevant sub-columns: `Launch Site`, `Lat(Latitude)`, `Long(Longitude)`, `class`
spacex_df = spacex_df[['Launch Site', 'Lat', 'Long', 'class']]
launch_sites_df = spacex_df.groupby(['Launch Site'], as_index=False).first()
launch_sites_df = launch_sites_df[['Launch Site', 'Lat', 'Long']]
launch_sites_df
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
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>CCAFS LC-40</td>
      <td>28.562302</td>
      <td>-80.577356</td>
    </tr>
    <tr>
      <th>1</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
    </tr>
    <tr>
      <th>2</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
    </tr>
    <tr>
      <th>3</th>
      <td>VAFB SLC-4E</td>
      <td>34.632834</td>
      <td>-120.610745</td>
    </tr>
  </tbody>
</table>
</div>



Above coordinates are just plain numbers that can not give you any intuitive insights about where are those launch sites. If you are very good at geography, you can interpret those numbers directly in your mind. If not, that's fine too. Let's visualize those locations by pinning them on a map.


We first need to create a folium `Map` object, with an initial center location to be NASA Johnson Space Center at Houston, Texas.



```python
# Start location is NASA Johnson Space Center
nasa_coordinate = [29.559684888503615, -95.0830971930759]
site_map = folium.Map(location=nasa_coordinate, zoom_start=10)
```

We could use `folium.Circle` to add a highlighted circle area with a text label on a specific coordinate. For example,



```python
# Create a blue circle at NASA Johnson Space Center's coordinate with a popup label showing its name
circle = folium.Circle(nasa_coordinate, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('NASA Johnson Space Center'))
# Create a blue circle at NASA Johnson Space Center's coordinate with a icon showing its name
marker = folium.map.Marker(
    nasa_coordinate,
    # Create an icon as a text label
    icon=DivIcon(
        icon_size=(20,20),
        icon_anchor=(0,0),
        html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'NASA JSC',
        )
    )
site_map.add_child(circle)
site_map.add_child(marker)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_86f12a059c14d8dac5ea8d53cd6868cc {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_86f12a059c14d8dac5ea8d53cd6868cc&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_86f12a059c14d8dac5ea8d53cd6868cc = L.map(
                &quot;map_86f12a059c14d8dac5ea8d53cd6868cc&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 10,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_0db3163b5f21fbeb7ab592bc2276a2e8 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_86f12a059c14d8dac5ea8d53cd6868cc);


            var circle_f7c0e3ab1d1eaafa08509d31304d9225 = L.circle(
                [29.559684888503615, -95.0830971930759],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_86f12a059c14d8dac5ea8d53cd6868cc);


        var popup_e89bb5b07bd42ccc6648eede3ef5a7df = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_a3032ed0c30a146d5e20632d763e34f3 = $(`&lt;div id=&quot;html_a3032ed0c30a146d5e20632d763e34f3&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;NASA Johnson Space Center&lt;/div&gt;`)[0];
            popup_e89bb5b07bd42ccc6648eede3ef5a7df.setContent(html_a3032ed0c30a146d5e20632d763e34f3);


        circle_f7c0e3ab1d1eaafa08509d31304d9225.bindPopup(popup_e89bb5b07bd42ccc6648eede3ef5a7df)
        ;




            var marker_c622c81dd2196fc517331634f4dbd71c = L.marker(
                [29.559684888503615, -95.0830971930759],
                {}
            ).addTo(map_86f12a059c14d8dac5ea8d53cd6868cc);


            var div_icon_e69629c2dab3e6cf370e49d80112d0e9 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eNASA JSC\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_c622c81dd2196fc517331634f4dbd71c.setIcon(div_icon_e69629c2dab3e6cf370e49d80112d0e9);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



and you should find a small yellow circle near the city of Houston and you can zoom-in to see a larger circle.


Now, let's add a circle for each launch site in data frame `launch_sites`


*TODO:*  Create and add `folium.Circle` and `folium.Marker` for each launch site on the site map


An example of folium.Circle:


`folium.Circle(coordinate, radius=1000, color='#000000', fill=True).add_child(folium.Popup(...))`


An example of folium.Marker:


`folium.map.Marker(coordinate, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'label', ))`



```python
LC40_Coordinates = launch_sites_df.iloc[0, [1,2]].tolist()
```


```python
LC400_coordinates = launch_sites_df.iloc[0, [1,2]].tolist()
SLC40_coordinates = launch_sites_df.iloc[1, [1,2]].tolist()
LC39A_coordinates = launch_sites_df.iloc[2, [1,2]].tolist()
SLC4E_coordinates = launch_sites_df.iloc[3, [1,2]].tolist()
```


```python
# Initial the map
site_map = folium.Map(location=nasa_coordinate, zoom_start=5)
# For each launch site, add a Circle object based on its coordinate (Lat, Long) values. In addition, add Launch site name as a popup label
LC400_circule = folium.Circle(LC400_coordinates, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('LC-400'))
LC400_marker = folium.map.Marker(LC400_coordinates, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'LC-400', ))
SLC40_circule = folium.Circle(SLC40_coordinates, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('SLC40'))
SLC40_marker = folium.map.Marker(SLC40_coordinates, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'SLC40', ))
LC39A_circule = folium.Circle(LC39A_coordinates, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('LC39A'))
LC39A_marker = folium.map.Marker(LC39A_coordinates, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'LC39A', ))
SLC4E_circule = folium.Circle(SLC4E_coordinates, radius=1000, color='#d35400', fill=True).add_child(folium.Popup('SLC4E'))
SLC4E_marker = folium.map.Marker(SLC4E_coordinates, icon=DivIcon(icon_size=(20,20),icon_anchor=(0,0), html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % 'SLC4E', ))
site_map.add_child(LC400_circule)
site_map.add_child(LC400_marker)
site_map.add_child(SLC40_circule)
site_map.add_child(SLC40_marker)
site_map.add_child(LC39A_circule)
site_map.add_child(LC39A_marker)
site_map.add_child(SLC4E_circule)
site_map.add_child(SLC4E_marker)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



The generated map with marked launch sites should look similar to the following:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_markers.png" />
</center>


Now, you can explore the map by zoom-in/out the marked areas
, and try to answer the following questions:

*   Are all launch sites in proximity to the Equator line?
*   Are all launch sites in very close proximity to the coast?

Also please try to explain your findings.


# Task 2: Mark the success/failed launches for each site on the map


Next, let's try to enhance the map by adding the launch outcomes for each site, and see which sites have high success rates.
Recall that data frame spacex_df has detailed launch records, and the `class` column indicates if this launch was successful or not



```python
spacex_df.tail(10)
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
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
      <th>class</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>47</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>48</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
    </tr>
    <tr>
      <th>49</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>50</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>51</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>52</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>53</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
    <tr>
      <th>54</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
    </tr>
    <tr>
      <th>55</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Next, let's create markers for all launch records.
If a launch was successful `(class=1)`, then we use a green marker and if a launch was failed, we use a red marker `(class=0)`


Note that a launch only happens in one of the four launch sites, which means many launch records will have the exact same coordinate. Marker clusters can be a good way to simplify a map containing many markers having the same coordinate.


Let's first create a `MarkerCluster` object



```python
marker_cluster = MarkerCluster()

```

*TODO:* Create a new column in `launch_sites` dataframe called `marker_color` to store the marker colors based on the `class` value



```python
marker_color = []
# Apply a function to check the value of `class` column
# If class=1, marker_color value will be green
# If class=0, marker_color value will be red
for n in spacex_df['class']:
    if n == 0:
        marker_color = 'red'
    else:
        marker_color = 'green'

marker_color
```




    'red'




```python
# Function to assign color to launch outcome
def assign_marker_color(launch_outcome):
    if launch_outcome == 1:
        return 'green'
    else:
        return 'red'
    
spacex_df['marker_color'] = spacex_df['class'].apply(assign_marker_color)
spacex_df.tail(10)
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
      <th>Launch Site</th>
      <th>Lat</th>
      <th>Long</th>
      <th>class</th>
      <th>marker_color</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>46</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>47</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>48</th>
      <td>KSC LC-39A</td>
      <td>28.573255</td>
      <td>-80.646895</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>49</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>50</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>51</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>52</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>53</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
    <tr>
      <th>54</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>1</td>
      <td>green</td>
    </tr>
    <tr>
      <th>55</th>
      <td>CCAFS SLC-40</td>
      <td>28.563197</td>
      <td>-80.576820</td>
      <td>0</td>
      <td>red</td>
    </tr>
  </tbody>
</table>
</div>



*TODO:* For each launch result in `spacex_df` data frame, add a `folium.Marker` to `marker_cluster`



```python
spacex_df['Lat'].astype(float)
spacex_df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 56 entries, 0 to 55
    Data columns (total 5 columns):
     #   Column        Non-Null Count  Dtype  
    ---  ------        --------------  -----  
     0   Launch Site   56 non-null     object 
     1   Lat           56 non-null     float64
     2   Long          56 non-null     float64
     3   class         56 non-null     int64  
     4   marker_color  56 non-null     object 
    dtypes: float64(2), int64(1), object(2)
    memory usage: 2.3+ KB



```python
# Add marker_cluster to current site_map
site_map.add_child(marker_cluster)

# for each row in spacex_df data frame
# create a Marker object with its coordinate
# and customize the Marker's icon property to indicate if this launch was successed or failed, 
# e.g., icon=folium.Icon(color='white', icon_color=row['marker_color']
icon = folium.Icon(color='white', icon_color=spacex_df['marker_color'])

for index, record in spacex_df.iterrows():
    # TODO: Create and add a Marker cluster to the site map
    marker = folium.Marker(location=[spacex_df.iloc[index]['Lat'],spacex_df.iloc[index]['Long']],icon = folium.Icon(color='white', icon_color =spacex_df.iloc[index]['marker_color']))
    # marker = folium.Marker(...)
    marker_cluster.add_child(marker)

site_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Your updated map may look like the following screenshots:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_cluster.png" />
</center>


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_cluster_zoomed.png" />
</center>


From the color-labeled markers in marker clusters, you should be able to easily identify which launch sites have relatively high success rates.


# TASK 3: Calculate the distances between a launch site to its proximities


Next, we need to explore and analyze the proximities of launch sites.


Let's first add a `MousePosition` on the map to get coordinate for a mouse over a point on the map. As such, while you are exploring the map, you can easily find the coordinates of any points of interests (such as railway)



```python
# Add Mouse Position to get the coordinate (Lat, Long) for a mouse over on the map
formatter = "function(num) {return L.Util.formatNum(num, 5);};"
mouse_position = MousePosition(
    position='topright',
    separator=' Long: ',
    empty_string='NaN',
    lng_first=False,
    num_digits=20,
    prefix='Lat:',
    lat_formatter=formatter,
    lng_formatter=formatter,
)

site_map.add_child(mouse_position)
site_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Now zoom in to a launch site and explore its proximity to see if you can easily find any railway, highway, coastline, etc. Move your mouse to these points and mark down their coordinates (shown on the top-left) in order to the distance to the launch site.


You can calculate the distance between two points on the map based on their `Lat` and `Long` values using the following method:



```python
from math import sin, cos, sqrt, atan2, radians

def calculate_distance(lat1, lon1, lat2, lon2):
    # approximate radius of earth in km
    R = 6373.0

    lat1 = radians(lat1)
    lon1 = radians(lon1)
    lat2 = radians(lat2)
    lon2 = radians(lon2)

    dlon = lon2 - lon1
    dlat = lat2 - lat1

    a = sin(dlat / 2)**2 + cos(lat1) * cos(lat2) * sin(dlon / 2)**2
    c = 2 * atan2(sqrt(a), sqrt(1 - a))

    distance = R * c
    return distance
```

*TODO:* Mark down a point on the closest coastline using MousePosition and calculate the distance between the coastline point and the launch site.



```python
# find coordinate of the closet coastline
# e.g.,: Lat: 28.56367  Lon: -80.57163
distance_coastline = calculate_distance(28.563197, -80.576820, 28.56346, -80.56802)
distance_coastline
```




    0.860186809123747



*TODO:* After obtained its coordinate, create a `folium.Marker` to show the distance



```python
# Create and add a folium.Marker on your selected closest coastline point on the map
# Display the distance between coastline point and launch site using the icon property 
# for example
distance_marker = folium.Marker(
    [28.56346, -80.56802],
    icon=DivIcon(
        icon_size=(20,20),
        icon_anchor=(0,0),
        html='<div style="font-size: 12; color:#d35400;"><b>%s</b></div>' % "{:10.2f} KM".format(distance_coastline),
    )
)
site_map.add_child(distance_marker)
site_map
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);


            var marker_fb56d2e6308d8b5d3cf2cccd1d75042b = L.marker(
                [28.56346, -80.56802],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_b273050330743c0da6ae619f85eccf91 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003e      0.86 KM\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_fb56d2e6308d8b5d3cf2cccd1d75042b.setIcon(div_icon_b273050330743c0da6ae619f85eccf91);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



*TODO:* Draw a `PolyLine` between a launch site to the selected coastline point



```python
# Create a `folium.PolyLine` object using the coastline coordinates and launch site coordinate
lines=folium.PolyLine([[28.56346, -80.56802], [28.563197,-80.576820]] , weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);


            var marker_fb56d2e6308d8b5d3cf2cccd1d75042b = L.marker(
                [28.56346, -80.56802],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_b273050330743c0da6ae619f85eccf91 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003e      0.86 KM\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_fb56d2e6308d8b5d3cf2cccd1d75042b.setIcon(div_icon_b273050330743c0da6ae619f85eccf91);


            var poly_line_dbacb90d1af9cdd33e1692322f365975 = L.polyline(
                [[28.56346, -80.56802], [28.563197, -80.57682]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



Your updated map with distance line should look like the following screenshot:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/launch_site_marker_distance.png" />
</center>


*TODO:* Similarly, you can draw a line betwee a launch site to its closest city, railway, highway, etc. You need to use `MousePosition` to find the their coordinates on the map first


A railway map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/railway.png" />
</center>


A highway map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/highway.png" />
</center>


A city map symbol may look like this:


<center>
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_3/images/city.png" />
</center>



```python
# Create a marker with distance to a closest city, railway, highway, etc.
# Draw a line between the marker to the launch site
lines=folium.PolyLine([[28.563197, -80.576820], [28.572,-80.58526]] , weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);


            var marker_fb56d2e6308d8b5d3cf2cccd1d75042b = L.marker(
                [28.56346, -80.56802],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_b273050330743c0da6ae619f85eccf91 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003e      0.86 KM\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_fb56d2e6308d8b5d3cf2cccd1d75042b.setIcon(div_icon_b273050330743c0da6ae619f85eccf91);


            var poly_line_dbacb90d1af9cdd33e1692322f365975 = L.polyline(
                [[28.56346, -80.56802], [28.563197, -80.57682]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_90bf779d72343b99cdfa21f5ed025cd6 = L.polyline(
                [[28.563197, -80.57682], [28.572, -80.58526]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
lines=folium.PolyLine([[28.563197, -80.576820], [28.56389,-80.57087]] , weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);


            var marker_fb56d2e6308d8b5d3cf2cccd1d75042b = L.marker(
                [28.56346, -80.56802],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_b273050330743c0da6ae619f85eccf91 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003e      0.86 KM\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_fb56d2e6308d8b5d3cf2cccd1d75042b.setIcon(div_icon_b273050330743c0da6ae619f85eccf91);


            var poly_line_dbacb90d1af9cdd33e1692322f365975 = L.polyline(
                [[28.56346, -80.56802], [28.563197, -80.57682]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_90bf779d72343b99cdfa21f5ed025cd6 = L.polyline(
                [[28.563197, -80.57682], [28.572, -80.58526]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_09c53f2aa85416fa535920d8d07f7d3f = L.polyline(
                [[28.563197, -80.57682], [28.56389, -80.57087]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
lines=folium.PolyLine([[28.563197, -80.576820], [28.40016,-80.6042]] , weight=1)
site_map.add_child(lines)
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><span style="color:#565656">Make this Notebook Trusted to load map: File -> Trust Notebook</span><iframe srcdoc="&lt;!DOCTYPE html&gt;
&lt;head&gt;    
    &lt;meta http-equiv=&quot;content-type&quot; content=&quot;text/html; charset=UTF-8&quot; /&gt;

        &lt;script&gt;
            L_NO_TOUCH = false;
            L_DISABLE_3D = false;
        &lt;/script&gt;

    &lt;style&gt;html, body {width: 100%;height: 100%;margin: 0;padding: 0;}&lt;/style&gt;
    &lt;style&gt;#map {position:absolute;top:0;bottom:0;right:0;left:0;}&lt;/style&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://code.jquery.com/jquery-1.12.4.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/js/bootstrap.min.js&quot;&gt;&lt;/script&gt;
    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/npm/leaflet@1.6.0/dist/leaflet.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap-theme.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://maxcdn.bootstrapcdn.com/font-awesome/4.6.3/css/font-awesome.min.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/Leaflet.awesome-markers/2.0.2/leaflet.awesome-markers.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/python-visualization/folium/folium/templates/leaflet.awesome.rotate.min.css&quot;/&gt;

            &lt;meta name=&quot;viewport&quot; content=&quot;width=device-width,
                initial-scale=1.0, maximum-scale=1.0, user-scalable=no&quot; /&gt;
            &lt;style&gt;
                #map_df2e0702092ec9217fce8745a6da8243 {
                    position: relative;
                    width: 100.0%;
                    height: 100.0%;
                    left: 0.0%;
                    top: 0.0%;
                }
            &lt;/style&gt;

    &lt;script src=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/leaflet.markercluster.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.css&quot;/&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdnjs.cloudflare.com/ajax/libs/leaflet.markercluster/1.1.0/MarkerCluster.Default.css&quot;/&gt;
    &lt;script src=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.js&quot;&gt;&lt;/script&gt;
    &lt;link rel=&quot;stylesheet&quot; href=&quot;https://cdn.jsdelivr.net/gh/ardhi/Leaflet.MousePosition/src/L.Control.MousePosition.min.css&quot;/&gt;
&lt;/head&gt;
&lt;body&gt;    

            &lt;div class=&quot;folium-map&quot; id=&quot;map_df2e0702092ec9217fce8745a6da8243&quot; &gt;&lt;/div&gt;

&lt;/body&gt;
&lt;script&gt;    

            var map_df2e0702092ec9217fce8745a6da8243 = L.map(
                &quot;map_df2e0702092ec9217fce8745a6da8243&quot;,
                {
                    center: [29.559684888503615, -95.0830971930759],
                    crs: L.CRS.EPSG3857,
                    zoom: 5,
                    zoomControl: true,
                    preferCanvas: false,
                }
            );





            var tile_layer_ff978f7a4fdf9d9af1bf90a9924a9021 = L.tileLayer(
                &quot;https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png&quot;,
                {&quot;attribution&quot;: &quot;Data by \u0026copy; \u003ca href=\&quot;http://openstreetmap.org\&quot;\u003eOpenStreetMap\u003c/a\u003e, under \u003ca href=\&quot;http://www.openstreetmap.org/copyright\&quot;\u003eODbL\u003c/a\u003e.&quot;, &quot;detectRetina&quot;: false, &quot;maxNativeZoom&quot;: 18, &quot;maxZoom&quot;: 18, &quot;minZoom&quot;: 0, &quot;noWrap&quot;: false, &quot;opacity&quot;: 1, &quot;subdomains&quot;: &quot;abc&quot;, &quot;tms&quot;: false}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var circle_0914db2ce50ad723f901fa5dcd442e55 = L.circle(
                [28.56230197, -80.57735648],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_735474b163077fa84abb6cee9d1bb43e = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_05832606a1c4e8ed1a3a455888b53b9c = $(`&lt;div id=&quot;html_05832606a1c4e8ed1a3a455888b53b9c&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC-400&lt;/div&gt;`)[0];
            popup_735474b163077fa84abb6cee9d1bb43e.setContent(html_05832606a1c4e8ed1a3a455888b53b9c);


        circle_0914db2ce50ad723f901fa5dcd442e55.bindPopup(popup_735474b163077fa84abb6cee9d1bb43e)
        ;




            var marker_259ddd09b3a2b85f252c402bcc6b600a = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_0041b226caf7858823576e2ededb971c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC-400\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_259ddd09b3a2b85f252c402bcc6b600a.setIcon(div_icon_0041b226caf7858823576e2ededb971c);


            var circle_d6eda5c8126a8bcfedcaba551086eae1 = L.circle(
                [28.56319718, -80.57682003],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_efe80597d9ea88d1e658e3e2870e0448 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_ec05023ec3624326ebcb25637ce766b7 = $(`&lt;div id=&quot;html_ec05023ec3624326ebcb25637ce766b7&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC40&lt;/div&gt;`)[0];
            popup_efe80597d9ea88d1e658e3e2870e0448.setContent(html_ec05023ec3624326ebcb25637ce766b7);


        circle_d6eda5c8126a8bcfedcaba551086eae1.bindPopup(popup_efe80597d9ea88d1e658e3e2870e0448)
        ;




            var marker_01a4cf45676ccba9ac6fd4f85106dd51 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_6f6a8f40702359da827e7365e87d7bc7 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC40\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_01a4cf45676ccba9ac6fd4f85106dd51.setIcon(div_icon_6f6a8f40702359da827e7365e87d7bc7);


            var circle_5f3d1e4ee8d2314c189b6365211d6ec1 = L.circle(
                [28.57325457, -80.64689529],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_5f664354e06160b88a6c727c914c9c6c = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_79f14aa5ad799826ca15f665606b812b = $(`&lt;div id=&quot;html_79f14aa5ad799826ca15f665606b812b&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;LC39A&lt;/div&gt;`)[0];
            popup_5f664354e06160b88a6c727c914c9c6c.setContent(html_79f14aa5ad799826ca15f665606b812b);


        circle_5f3d1e4ee8d2314c189b6365211d6ec1.bindPopup(popup_5f664354e06160b88a6c727c914c9c6c)
        ;




            var marker_854c8f37bda2c072f0775230229a7c4b = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eLC39A\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_854c8f37bda2c072f0775230229a7c4b.setIcon(div_icon_69ea862d60bfefac8b1c6ac5a3ce0a22);


            var circle_2d8ba9237a295429fa7f65d9f0a48829 = L.circle(
                [34.63283416, -120.6107455],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#d35400&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: true, &quot;fillColor&quot;: &quot;#d35400&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;opacity&quot;: 1.0, &quot;radius&quot;: 1000, &quot;stroke&quot;: true, &quot;weight&quot;: 3}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


        var popup_9e28f66899acb10c5ab1e04ed8270904 = L.popup({&quot;maxWidth&quot;: &quot;100%&quot;});


            var html_7d07e03db1cebb9950639eeee346c1b5 = $(`&lt;div id=&quot;html_7d07e03db1cebb9950639eeee346c1b5&quot; style=&quot;width: 100.0%; height: 100.0%;&quot;&gt;SLC4E&lt;/div&gt;`)[0];
            popup_9e28f66899acb10c5ab1e04ed8270904.setContent(html_7d07e03db1cebb9950639eeee346c1b5);


        circle_2d8ba9237a295429fa7f65d9f0a48829.bindPopup(popup_9e28f66899acb10c5ab1e04ed8270904)
        ;




            var marker_f889672213440eb9be81d25c98a99cbf = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_95496c9efdcdaa089e7f90c00c56e74c = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003eSLC4E\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_f889672213440eb9be81d25c98a99cbf.setIcon(div_icon_95496c9efdcdaa089e7f90c00c56e74c);


            var marker_cluster_166113a13a079decc18ee2542f0da1d1 = L.markerClusterGroup(
                {}
            );
            map_df2e0702092ec9217fce8745a6da8243.addLayer(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var marker_c0e1e0a11d2d08331f99e53f77ade7e5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_64704663c22e90121b5a4574e9eaf4b7 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c0e1e0a11d2d08331f99e53f77ade7e5.setIcon(icon_64704663c22e90121b5a4574e9eaf4b7);


            var marker_0039c1f125883027d86b3a727ef7ed1d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_732c9f3dd1c69ee66710ff75c29695a9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_0039c1f125883027d86b3a727ef7ed1d.setIcon(icon_732c9f3dd1c69ee66710ff75c29695a9);


            var marker_dbb12e147b760a6c4ee242ad01d09a5e = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_816d9a469956e8b9571289e2d01972f1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dbb12e147b760a6c4ee242ad01d09a5e.setIcon(icon_816d9a469956e8b9571289e2d01972f1);


            var marker_6d0aac8787573d4ae197ba1489184d60 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ed7ba77f9b6c7e7f4fb8de98ecef2282 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6d0aac8787573d4ae197ba1489184d60.setIcon(icon_ed7ba77f9b6c7e7f4fb8de98ecef2282);


            var marker_43386bedad0f9309d316156c773c1eeb = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2dc354698c3f565c75da0458d5e7c654 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_43386bedad0f9309d316156c773c1eeb.setIcon(icon_2dc354698c3f565c75da0458d5e7c654);


            var marker_ae0e80f26954a726bd2ca15b4affd897 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6ada9d57dce9bca1bcfaba6d82c5c65a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ae0e80f26954a726bd2ca15b4affd897.setIcon(icon_6ada9d57dce9bca1bcfaba6d82c5c65a);


            var marker_7f580a5d57c4d9e89a14448fb3684b5b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_af1f02607cd67ac7d1954a0662f1cdfd = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7f580a5d57c4d9e89a14448fb3684b5b.setIcon(icon_af1f02607cd67ac7d1954a0662f1cdfd);


            var marker_fd3d2c37f20e21979275b77f1cbacef3 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_71e8d2e8e3ffd6fa7d50dd5528412ec6 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fd3d2c37f20e21979275b77f1cbacef3.setIcon(icon_71e8d2e8e3ffd6fa7d50dd5528412ec6);


            var marker_f80308a6978aa028d4c167e397ad1e9c = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_3281629486460f46f1f0b9c4c1e1469f = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_f80308a6978aa028d4c167e397ad1e9c.setIcon(icon_3281629486460f46f1f0b9c4c1e1469f);


            var marker_653d6bb4d885b5c65735124ca5cd13aa = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_79d977824b4297ea7b7698a43437af80 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_653d6bb4d885b5c65735124ca5cd13aa.setIcon(icon_79d977824b4297ea7b7698a43437af80);


            var marker_4ec663d6955862b6f00faaf0bfa49760 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f1e1fe3142335127a8493e2a4358243 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4ec663d6955862b6f00faaf0bfa49760.setIcon(icon_1f1e1fe3142335127a8493e2a4358243);


            var marker_e8ee5059807d87556ffded6dd618e1cc = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_1f73c890f056e3e61df06bbe2e29a0b9 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e8ee5059807d87556ffded6dd618e1cc.setIcon(icon_1f73c890f056e3e61df06bbe2e29a0b9);


            var marker_9e38a5e947790e5eeceede8b86ae9cba = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_612ed9736992c42a6417d371e37ff783 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9e38a5e947790e5eeceede8b86ae9cba.setIcon(icon_612ed9736992c42a6417d371e37ff783);


            var marker_8841fd149b7cf66479af00b078d4057d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7021b72d7a411a8076bba4e31bc815ed = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_8841fd149b7cf66479af00b078d4057d.setIcon(icon_7021b72d7a411a8076bba4e31bc815ed);


            var marker_142a46ac5b95f2d06b05b8f2c4e63836 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_848f5dee216c4ed3e1fd78b06ae63323 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_142a46ac5b95f2d06b05b8f2c4e63836.setIcon(icon_848f5dee216c4ed3e1fd78b06ae63323);


            var marker_11c177e493536b8ef6983930ecac5542 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_936465ba0acf16fce01ae3c010acd05a = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_11c177e493536b8ef6983930ecac5542.setIcon(icon_936465ba0acf16fce01ae3c010acd05a);


            var marker_c85d5692c7ef6f3627adfd208c092833 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_c5767bdfc7393bccdec7368eda62c803 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c85d5692c7ef6f3627adfd208c092833.setIcon(icon_c5767bdfc7393bccdec7368eda62c803);


            var marker_6e538ecb5e015b9e69b5573cc288cc7b = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad980bf2cad6fc15b4f7630f3198071 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e538ecb5e015b9e69b5573cc288cc7b.setIcon(icon_2ad980bf2cad6fc15b4f7630f3198071);


            var marker_53cd0e7a97f4c1d3b749b4aed41ad887 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_ec73d1eac3f5c15d4e549e25fe0da586 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_53cd0e7a97f4c1d3b749b4aed41ad887.setIcon(icon_ec73d1eac3f5c15d4e549e25fe0da586);


            var marker_b9e429f2b41d1bfe011f741e2f340402 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_6b50b3535afd6ea8a4e231bd56cb397e = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b9e429f2b41d1bfe011f741e2f340402.setIcon(icon_6b50b3535afd6ea8a4e231bd56cb397e);


            var marker_d741e7956e3884c4d8dfb9ee503c8f89 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_e6b93450d87ac42a1d104e752e3c3c97 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d741e7956e3884c4d8dfb9ee503c8f89.setIcon(icon_e6b93450d87ac42a1d104e752e3c3c97);


            var marker_7a07e0b14fc3b87947128539614fe43d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4f25482cccbee0915f307790cff8fe14 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7a07e0b14fc3b87947128539614fe43d.setIcon(icon_4f25482cccbee0915f307790cff8fe14);


            var marker_a2dccc23c8974bfd7deecbd27044bdbd = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_90f964f2b220639e37bdd56d1d00f948 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_a2dccc23c8974bfd7deecbd27044bdbd.setIcon(icon_90f964f2b220639e37bdd56d1d00f948);


            var marker_3528a2a9a10a296a3bcc871f9212d3b5 = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_10f80e35b485d6c5ccae1d78171c5fac = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3528a2a9a10a296a3bcc871f9212d3b5.setIcon(icon_10f80e35b485d6c5ccae1d78171c5fac);


            var marker_20eb9cc80ff83925049937721d52f91f = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_7e185a88cae0ecdb98a45460c7cb2b20 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_20eb9cc80ff83925049937721d52f91f.setIcon(icon_7e185a88cae0ecdb98a45460c7cb2b20);


            var marker_5f1a24738428529b439c174e34e7e58d = L.marker(
                [28.56230197, -80.57735648],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_08ba6b12891a3c3897f411124f5a9329 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_5f1a24738428529b439c174e34e7e58d.setIcon(icon_08ba6b12891a3c3897f411124f5a9329);


            var marker_cc683a3ba05780b28da7668ea1db2bca = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_571e6feb8985902fe19523220fd28e83 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_cc683a3ba05780b28da7668ea1db2bca.setIcon(icon_571e6feb8985902fe19523220fd28e83);


            var marker_e99e0d7b39de636311d455dd76bec2a9 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_247017e824689cbf6c3405f62641b699 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e99e0d7b39de636311d455dd76bec2a9.setIcon(icon_247017e824689cbf6c3405f62641b699);


            var marker_b44954b8ff45fc0fe7a9cab556c15fe2 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_9843da7db898aee85439eeb3e59a01e0 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b44954b8ff45fc0fe7a9cab556c15fe2.setIcon(icon_9843da7db898aee85439eeb3e59a01e0);


            var marker_bd80620c2d9802351f37362e85027e35 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_18d1e49ac47f9288c8668dc5a6c73c04 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_bd80620c2d9802351f37362e85027e35.setIcon(icon_18d1e49ac47f9288c8668dc5a6c73c04);


            var marker_132a8141fb6add8bd8ef2fed4cb80cee = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2aa6252d8e4b8864ab4ee3216f145497 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_132a8141fb6add8bd8ef2fed4cb80cee.setIcon(icon_2aa6252d8e4b8864ab4ee3216f145497);


            var marker_15c4f3622e28bc985ab61f1c9a725c37 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0fc09112057efa1cdfcbab0556662253 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_15c4f3622e28bc985ab61f1c9a725c37.setIcon(icon_0fc09112057efa1cdfcbab0556662253);


            var marker_936ac9441d5933bab67e05fd82b5ff50 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ad3f3b7004b3a3a16649c54b74bb999 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_936ac9441d5933bab67e05fd82b5ff50.setIcon(icon_2ad3f3b7004b3a3a16649c54b74bb999);


            var marker_dfdc1f283a8a0ea1366667581c855ff4 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_8a1ce95324b17adf665ed3c85d751801 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_dfdc1f283a8a0ea1366667581c855ff4.setIcon(icon_8a1ce95324b17adf665ed3c85d751801);


            var marker_ef5fed334e18dcbbd466a5b16d4b0166 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_466f6caa72d447d421da05be0eeac146 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ef5fed334e18dcbbd466a5b16d4b0166.setIcon(icon_466f6caa72d447d421da05be0eeac146);


            var marker_4830e071925314192736285a6b3ea552 = L.marker(
                [34.63283416, -120.6107455],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fe0bd10468fa147c662a20cba4eb69c4 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4830e071925314192736285a6b3ea552.setIcon(icon_fe0bd10468fa147c662a20cba4eb69c4);


            var marker_3efadcd428ba91fc6eac1fdedbc37d5a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f80c6dfbd66033fb9f3010cbf4f3af44 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_3efadcd428ba91fc6eac1fdedbc37d5a.setIcon(icon_f80c6dfbd66033fb9f3010cbf4f3af44);


            var marker_994371ba7fda4fb101ca7a995e61b799 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2617e329809f8ad172404df206dc6297 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_994371ba7fda4fb101ca7a995e61b799.setIcon(icon_2617e329809f8ad172404df206dc6297);


            var marker_9f82bdcf2dae41ee8b6be8d936846672 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2525b6e213e78f34408b5b1badbb0b87 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_9f82bdcf2dae41ee8b6be8d936846672.setIcon(icon_2525b6e213e78f34408b5b1badbb0b87);


            var marker_6b3ee18ea8c08b5ecdf457cbf1610ec5 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_5b107e4eab6ecc139e28fd90567121bf = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6b3ee18ea8c08b5ecdf457cbf1610ec5.setIcon(icon_5b107e4eab6ecc139e28fd90567121bf);


            var marker_4c486ce7ca58ecc0788fabf4f8abb5b0 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2bda6b11a7626e88eb3f8420fab06dca = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_4c486ce7ca58ecc0788fabf4f8abb5b0.setIcon(icon_2bda6b11a7626e88eb3f8420fab06dca);


            var marker_14897c44325f7efcb04776dc477c38fc = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_30ec6cd981ae9579b2ce2b584a378c07 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_14897c44325f7efcb04776dc477c38fc.setIcon(icon_30ec6cd981ae9579b2ce2b584a378c07);


            var marker_af1b1d801237004a6c74690b80713989 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_456f0511e167f1566256473adfc67e62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_af1b1d801237004a6c74690b80713989.setIcon(icon_456f0511e167f1566256473adfc67e62);


            var marker_6e7d47e8c812807f093ea096defc3ea1 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_f71a703a63ddc239c414f9b45d88ba57 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e7d47e8c812807f093ea096defc3ea1.setIcon(icon_f71a703a63ddc239c414f9b45d88ba57);


            var marker_e63a1e3aaa060297efdf42ca2de3e2d6 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_99ea6524b3172d72f2c030ae2e7611c1 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_e63a1e3aaa060297efdf42ca2de3e2d6.setIcon(icon_99ea6524b3172d72f2c030ae2e7611c1);


            var marker_34dc5e840f53dbfd4608a9ea8277e72a = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_858bcb8c12f7507ffdf225315c6c09c3 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_34dc5e840f53dbfd4608a9ea8277e72a.setIcon(icon_858bcb8c12f7507ffdf225315c6c09c3);


            var marker_d9678ea3772d53f636723311df03e683 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_94f2cfeb85c314dd3e537a4ceef5f622 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d9678ea3772d53f636723311df03e683.setIcon(icon_94f2cfeb85c314dd3e537a4ceef5f622);


            var marker_7976cce9e7409fedf2f476d723b492a2 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_0d1f9718dcce187627a066e5b1dcee41 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_7976cce9e7409fedf2f476d723b492a2.setIcon(icon_0d1f9718dcce187627a066e5b1dcee41);


            var marker_fa7b95abb9a99341f24bc31ed634c969 = L.marker(
                [28.57325457, -80.64689529],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_2ce03539a8052b725282e253213b0c62 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_fa7b95abb9a99341f24bc31ed634c969.setIcon(icon_2ce03539a8052b725282e253213b0c62);


            var marker_ad369ac3f95096f99a8dcc79e1c742fd = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_749bd66e2d98bf80fa8e3c0760f4854d = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_ad369ac3f95096f99a8dcc79e1c742fd.setIcon(icon_749bd66e2d98bf80fa8e3c0760f4854d);


            var marker_6e852bfa1371debd0581485f1a5a7a6e = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_fea392f98b982df6c6c8ad332440bf96 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_6e852bfa1371debd0581485f1a5a7a6e.setIcon(icon_fea392f98b982df6c6c8ad332440bf96);


            var marker_b812c3e603d0852c652f0b5a36fb449b = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_afc8ae9398c094ec914422a2fceb3797 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_b812c3e603d0852c652f0b5a36fb449b.setIcon(icon_afc8ae9398c094ec914422a2fceb3797);


            var marker_c8dc28273165caf2588eb753dcb0fa47 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_4cd654e63a968dd86108ad8c90b800d2 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_c8dc28273165caf2588eb753dcb0fa47.setIcon(icon_4cd654e63a968dd86108ad8c90b800d2);


            var marker_224cf7fd94e916060c1592436c3c28db = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_b913df9bd7264c646d3d4154411b63ee = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_224cf7fd94e916060c1592436c3c28db.setIcon(icon_b913df9bd7264c646d3d4154411b63ee);


            var marker_efeecb11df8d58e4a4317e426166edce = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_d7fb13cb549ca0ec7a1d772eb24bf334 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;green&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_efeecb11df8d58e4a4317e426166edce.setIcon(icon_d7fb13cb549ca0ec7a1d772eb24bf334);


            var marker_d14771e73798fd169c82cdbca1741916 = L.marker(
                [28.56319718, -80.57682003],
                {}
            ).addTo(marker_cluster_166113a13a079decc18ee2542f0da1d1);


            var icon_39e425577030b54eb6175ad4bb108698 = L.AwesomeMarkers.icon(
                {&quot;extraClasses&quot;: &quot;fa-rotate-0&quot;, &quot;icon&quot;: &quot;info-sign&quot;, &quot;iconColor&quot;: &quot;red&quot;, &quot;markerColor&quot;: &quot;white&quot;, &quot;prefix&quot;: &quot;glyphicon&quot;}
            );
            marker_d14771e73798fd169c82cdbca1741916.setIcon(icon_39e425577030b54eb6175ad4bb108698);


            var mouse_position_ad2f85c91b62cac079468aef7e315631 = new L.Control.MousePosition(
                {&quot;emptyString&quot;: &quot;NaN&quot;, &quot;lngFirst&quot;: false, &quot;numDigits&quot;: 20, &quot;position&quot;: &quot;topright&quot;, &quot;prefix&quot;: &quot;Lat:&quot;, &quot;separator&quot;: &quot; Long: &quot;}
            );
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;latFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            mouse_position_ad2f85c91b62cac079468aef7e315631.options[&quot;lngFormatter&quot;] =
                function(num) {return L.Util.formatNum(num, 5);};;
            map_df2e0702092ec9217fce8745a6da8243.addControl(mouse_position_ad2f85c91b62cac079468aef7e315631);


            var marker_fb56d2e6308d8b5d3cf2cccd1d75042b = L.marker(
                [28.56346, -80.56802],
                {}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var div_icon_b273050330743c0da6ae619f85eccf91 = L.divIcon({&quot;className&quot;: &quot;empty&quot;, &quot;html&quot;: &quot;\u003cdiv style=\&quot;font-size: 12; color:#d35400;\&quot;\u003e\u003cb\u003e      0.86 KM\u003c/b\u003e\u003c/div\u003e&quot;, &quot;iconAnchor&quot;: [0, 0], &quot;iconSize&quot;: [20, 20]});
            marker_fb56d2e6308d8b5d3cf2cccd1d75042b.setIcon(div_icon_b273050330743c0da6ae619f85eccf91);


            var poly_line_dbacb90d1af9cdd33e1692322f365975 = L.polyline(
                [[28.56346, -80.56802], [28.563197, -80.57682]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_90bf779d72343b99cdfa21f5ed025cd6 = L.polyline(
                [[28.563197, -80.57682], [28.572, -80.58526]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_09c53f2aa85416fa535920d8d07f7d3f = L.polyline(
                [[28.563197, -80.57682], [28.56389, -80.57087]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);


            var poly_line_0e31239aacd99f342aeb5934a1288823 = L.polyline(
                [[28.563197, -80.57682], [28.40016, -80.6042]],
                {&quot;bubblingMouseEvents&quot;: true, &quot;color&quot;: &quot;#3388ff&quot;, &quot;dashArray&quot;: null, &quot;dashOffset&quot;: null, &quot;fill&quot;: false, &quot;fillColor&quot;: &quot;#3388ff&quot;, &quot;fillOpacity&quot;: 0.2, &quot;fillRule&quot;: &quot;evenodd&quot;, &quot;lineCap&quot;: &quot;round&quot;, &quot;lineJoin&quot;: &quot;round&quot;, &quot;noClip&quot;: false, &quot;opacity&quot;: 1.0, &quot;smoothFactor&quot;: 1.0, &quot;stroke&quot;: true, &quot;weight&quot;: 1}
            ).addTo(map_df2e0702092ec9217fce8745a6da8243);

&lt;/script&gt;" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



After you plot distance lines to the proximities, you can answer the following questions easily:

*   Are launch sites in close proximity to railways?
*   Are launch sites in close proximity to highways?
*   Are launch sites in close proximity to coastline?
*   Do launch sites keep certain distance away from cities?

Also please try to explain your findings.


# Next Steps:

Now you have discovered many interesting insights related to the launch sites' location using folium, in a very interactive way. Next, you will need to build a dashboard using Ploty Dash on detailed launch records.


## Authors


[Yan Luo](https://www.linkedin.com/in/yan-luo-96288783/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01)


### Other Contributors


Joseph Santarcangelo


## Change Log


| Date (YYYY-MM-DD) | Version | Changed By | Change Description          |
| ----------------- | ------- | ---------- | --------------------------- |
| 2021-05-26        | 1.0     | Yan        | Created the initial version |


Copyright © 2021 IBM Corporation. All rights reserved.

