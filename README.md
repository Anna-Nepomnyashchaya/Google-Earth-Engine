# Google-Earth-Engine
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
.filterBounds(bounds) // Фильтруем по области интереса 
.filterDate(start,finish) // Фильтруем по датам 
.filter(ee.Filter.lt('CLOUD_COVERAGE_ASSESSMENT',9)).map(addNDVI) ; // Фильтруем по облачности 
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
