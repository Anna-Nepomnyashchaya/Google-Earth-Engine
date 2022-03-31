# Google Earth Engine
---------------------
We use machine learning when working with geodata. Geologists are searching for minerals such as sand for construction. Sand lies at the bottom of thermokarst lakes. With the help of Google Earth Engine and machine learning based on satellite imagery, we created a training sample by selecting a certain number of thermokarst lakes manually. Then a test sample of a larger area was selected on which the algorithm selected all the lakes. Thus, a map was created for geologists that will help them in the search for minerals, that is, sand at the bottom of thermokarst lakes.

Python and JavaScript client libraries for calling the Google Earth Engine API

-   [Earth Engine Homepage](https://earthengine.google.com/)
-   [Web Code Editor](https://code.earthengine.google.com/)
-   [Python
    Installation](https://developers.google.com/earth-engine/python_install)

Example screenshot and the corresponding Code Editor JavaScript code:

![2022-03-31_19-40-21](https://user-images.githubusercontent.com/36707478/161106714-54542ec5-1deb-4584-8a7a-08ce7edebd25.png)

```javascript
var bounds = /* color: #00ffff */ee.Geometry.Polygon( 
[[[96.48193359375, 72.65958846878621], 
[96.13037109375, 71.30079291637452], 
[98.41809151958319, 71.28725578372836], 
[100.87646484375, 71.27259471233448], 
[101.14013671875, 72.67268149675316]]]); 
var polygons=lake.merge(land); 
Map.addLayer(polygons,{}, 'Dataset for training'); 
function addNDVI (image){ 
var ndvi = image.normalizedDifference(['B8A','B4']).rename('NDVI'); 
return image.addBands(ndvi); 
} 
var start = ee.Date('2016-06-01'); 
var finish = ee.Date('2016-08-31'); 
var img = ee.ImageCollection('COPERNICUS/S2')
.filterBounds(bounds) // Filter by the required area
.filterDate(start,finish) // Filtering by dates 
.filter(ee.Filter.lt('CLOUD_COVERAGE_ASSESSMENT',9)).map(addNDVI) ; // Filtering by clouds
var image = ee.Image(img.sort('CLOUD_COVERAGE_ASSESSMENT') .min()).clip(bounds); 
var visParams = {bands: ['B4', 'B3', 'B2'], gamma: 2, min:300, max:3000}; 
Map.centerObject(bounds, 8); 
Map.addLayer(image,visParams,'Image'); 
var bands = ['B1','B2', 'B3', 'B4', 'B5', 'B6', 'B7', 'B8', 'B8A', 'B10', 'B11', 'B12' ,'NDVI']; 
var imag = image.select(bands); 
var polygons = polygons; 
var training = imag.sampleRegions({ 
// Get the sample from the polygons FeatureCollection. 
collection: polygons, 
// Keep this list of properties from the polygons. 
properties: ['class'], 
// Set the scale to get Landsat pixels in the polygons. 
scale: 100 
}); 
var classifier = ee.Classifier.randomForest({ 
numberOfTrees: 10, 
}); 
var trained = classifier.train(training, 'class', bands); 
var classified = imag.classify(trained); 
var palette = [ 
'pink', 
'blue' 
]; 
Map.addLayer(classified, {min: 0, max: 1, palette: palette}, 'Classified');
```
