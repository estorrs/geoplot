# geoscatter

**A python API for adding layers of points and heatmaps to a map.**

***

## Table of Contents
1. [Installation](#installation)
1. [Usage](#usage)
	1. [Setup](#setup)
	1. [Adding Points](#adding-points)
	1. [Adding Heatmaps](#adding-heatmaps)
	1. [Adding Multiple Datasets](#adding-multiple-datasets)
1. [Documentation](#documentation)
1. [License](#license)
1. [Thanks](#thanks)
	
***

### Installation

You will need [pip](https://packaging.python.org/installing/) on your machine. 

At the command line 
```bash
pip install geoscatter
```
***

### Usage
1. [Setup](#setup)
1. [Adding Points](#adding-points)
1. [Adding Heatmaps](#adding-heatmaps)
1. [Adding Multiple Datasets](#adding-multiple-datasets)




#### Setup

First import the GeoScatter and GeoPoint classes from geoscatter

```python
from geoscatter import GeoScatter
from geoscatter import GeoPoint
```

To use geoscatter, you will need a Google Static Maps API key.  It's quick and free!  Just go [here](https://developers.google.com/maps/documentation/static-maps/) and click *Get A Key* in the top right corner of the page. 
```python
googleMapsApiKey = "INSERT YOUR API KEY HERE!!"
```

To begin, we define our map.  We do so by specifying the center of the map by creating a GeoPoint containing the latitude and longitude of the center coordinate (Columbia, Mo in this case).  Additionally, we can specify a zoom level between 1 and 20 and the size of the map in pixels.  A quick note on map size, if a size below 150 or above 600, geoscatter may not work as intended. Also you can choose to get a map with or without labels.   
```python
cLat = 38.536178
cLon = -92.302498
center = GeoPoint(cLat, cLon)

gs = GeoScatter(googleMapsApiKey, center, zoom=3, imgSize=600, mapLabels=False)
```

#### Adding Points



Before we add some points to our map, I'd like to take a moment to talk about the data I'll be using in these examples.  The datasets are the starting location of all US tornado tracks since 1950 (National Weather Service) and all *"significant"* global earthquakes since 2500 B.C. (USGS). 

There are multiple ways we can add points to our map.  First we can add points by file.  The file must have two fields.  This first field is the point latitude and the second is the longitude.  The fields can be separated by either a comma or tab.  So both latitude,longitude and latitude\tlongitude are acceptable. 

Lets plot all tornadoes in the United States since 1950. 
```python
gs.addPoints("../example_data/tornado_locations.csv", s=.5, alpha=.4, c='r', marker='o')
gs.showMap()
```
![all tornadoes](/images/tornadoes/all_tornadoes.png)

We can also change point shape, size, color, and transparency.  It's messy but here we plot the tornadoes in Missouri as 10 pixel wide blue triangles with no transparency.  For more about [marker](http://matplotlib.org/api/markers_api.html) and [color](https://matplotlib.org/api/colors_api.html) see matplotlib's documentation.
```python
gs = GeoScatter(googleMapsApiKey, center, zoom=7, imgSize = 600, mapLabels=False)
gs.addPoints("../example_data/tornado_locations.csv", s=10, alpha=1.0, c='b', marker='v')
gs.showMap()
```
![missouri tornadoes](/images/tornadoes/missouri_tornadoes_notransparency.png)

What if we wan't points in the same dataset to have different sizes?  We can do that by using a file with three fields.

The first two fields are the points latitude and longitude.  The third field is the point size.  The fields can be separated by commas or tabs. 

Here we plot the same tornadoes, but the size of the point corresponds to the number of deaths caused by the tornado.  
```python
gs = GeoScatter(googleMapsApiKey, center, zoom=7, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/tornadoes_deaths.csv", s=1, alpha=.4, c='r', marker='o')
gs.showMap()
```
![missouri torandoes triangles](/images/tornadoes/missouri_tornadoes_deaths.png)

When we add points this way, the size parameter becomes a scalar.  So for instance, if we want to make the points 3X larger. 

```python 
gs = GeoScatter(googleMapsApiKey, center, zoom=7, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/tornadoes_deaths.csv", s=3, alpha=.4, c='r', marker='o')
gs.showMap()
```
![tornado deaths](/images/tornadoes/missouri_tornadoes_deaths_s3.png)

Lastly, we can manually add the points ourselves by creating a list of GeoPoints. GeoPoints have three attributes: latitude, longitude, and magnitude.  

```python
pts = []
inputFile = file("../example_data/tornadoes_deaths.csv", "r")
for line in inputFile: 
    line = line.strip()
    pieces = line.split(",")
    newPoint = GeoPoint(float(pieces[0]), float(pieces[1]))
    newPoint.magnitude = float(pieces[2])
    pts.append(newPoint)
    
gs = GeoScatter(googleMapsApiKey, center, zoom=7, imgSize=600, mapLabels=True)
gs.addPoints(pts, s=1, alpha=.4, c='r', marker='o')
gs.showMap()
```
![tornado manual](/images/tornadoes/missouri_tornadoes_deaths.png)


#### Adding Heatmaps

geoscatter also supports heatmaps.  

First we create a map of all *"significant"* earthquakes in the past 4500 years.  

```python
cLat = 0
cLon = 0
center = GeoPoint(cLat, cLon)
gs = GeoScatter(googleMapsApiKey, center, zoom=1, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='#FFFF00', marker='s')
gs.showMap()
```
![earthquakes](/images/earthquakes/all_earthquakes.png)

Now we add a heatmap.  A brief warning that when the quantity of points starts to get into the thousands you may have to wait a minute or two for the heatmap to generate.  And even longer if you are using more points than that. 
```python
cLat = 0
cLon = 0
center = GeoPoint(cLat, cLon)
gs = GeoScatter(googleMapsApiKey, center, zoom=1, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='#FFFF00', marker='s')
gs.addHeatmap(colorscheme="coolwarm", bw=25.0, alpha=.4)
gs.showMap()
```
![heatmap](/images/earthquakes/all_earthquakes_heatmap.png)

The heatmap bandwidth can be adjusted. Bandwidth controls the *"smoothness"* of the heatmap.  A larger bandwidth will result in a smoother heatmap.  

Here we zoom in on California and use a bandwidth of 10.0. 

```python
cLat = 39.140328
cLon = -120.034881
center = GeoPoint(cLat, cLon)
gs = GeoScatter(googleMapsApiKey, center, zoom=5, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='#FFFF00', marker='s')
gs.addHeatmap(colorscheme="coolwarm", bw=10.0, alpha=.4)
gs.showMap()
```
![bw 10](/images/earthquakes/california_earthquakes_heatmap_bw10.png)

And a bandwidth of 100.0. 

```python
gs = GeoScatter(googleMapsApiKey, center, zoom=5, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='#FFFF00', marker='s')
gs.addHeatmap(colorscheme="coolwarm", bw=100.0, alpha=.4)
gs.showMap()
```
![bw 100](/images/earthquakes/california_earthquakes_heatmap_bw100.png)

We can also alter the colorscheme. For more on [colorschemes](http://matplotlib.org/examples/color/colormaps_reference.html) visit matplotlib's documentation. 

```python
gs = GeoScatter(googleMapsApiKey, center, zoom=5, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='#FFFF00', marker='s')
gs.addHeatmap(colorscheme="Reds", bw=25.0, alpha=.4)
gs.showMap()
```
![reds](/images/earthquakes/california_earthquake_heatmap_reds.png)

We can also display only the heatmap by setting point size to 0

```python
gs = GeoScatter(googleMapsApiKey, center, zoom=5, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=0, alpha=1.0, c='#FFFF00', marker='s')
gs.addHeatmap(colorscheme="coolwarm", bw=25.0, alpha=.4)
gs.showMap()
```
![nopoints](/images/earthquakes/california_earthquakes_heatmap_nopoints.png)

#### Adding Multiple Datasets

geoscatter allows for visualization of multiple sets of points.  Here we plot both earthquakes (blue squares) and tornadoes(red triangles).  

```python 
cLat = 36.157467
cLon = -120.862654
center = GeoPoint(cLat, cLon)
gs = GeoScatter(googleMapsApiKey, center, zoom=6, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='b', marker='s')
gs.addPoints("../example_data/tornado_locations.csv", s=5, alpha=1.0, c='r', marker='v')
gs.showMap()
```
![mult datasets](/images/earthquakes/california_earthquakes_tornadoes.png)

We can also do separate heatmaps. 

```python
gs = GeoScatter(googleMapsApiKey, center, zoom=6, imgSize=600, mapLabels=False)
gs.addPoints("../example_data/earthquake_locations.csv", s=5, alpha=1.0, c='b', marker='s')
gs.addHeatmap(colorscheme="Blues", bw=30.0, alpha=.4)
gs.addPoints("../example_data/tornado_locations.csv", s=5, alpha=1.0, c='r', marker='v')
gs.addHeatmap(colorscheme="Reds", bw=30.0, alpha=.4)
gs.showMap()
```
![mult heatmapts](/images/earthquakes/california_earthquakes_tornadoes_heatmap.png)

***

### Documentation

See [wiki](https://github.com/estorrs/geoscatter/wiki). 

***

### License

Available under the [MIT license](LICENSE.txt). 

***

### Thanks

Thanks to [Kevin Scannell](https://borel.slu.edu) and [Erin Chambers](http://cs.slu.edu/~chambers/) for their help on this project.  

And of course the contributors to the APIs used in this project. 
[matplotlib](https://matplotlib.org)
[scipy](https://scipy.org)
[mercantile](https://github.com/mapbox/mercantile)
[sklearn](https://sk-learn.org)



