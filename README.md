# Fork notes

This fork contains some performance tweaks for large datasets (100k+ points). It does not match upstream's full featureset.

## Changes

- `addDataPoint()` removed. Redraw is too expensive on big datasets, heatmaps should be considered immutable.

- `setDataSet()` stores data y,x to reduce the iterations in the `drawAlpha()` loop. The assumption is that sites are taller than they are wide.

- `drawAlpha()` operates against the entire dataset. It does not draw individual paths for each point anymore. Instead it draws one path per iterated row.
  This does change the look of the resulting heatmap *slighly*.

- `colorize()` has been split into an outer (still called `colorize()`) and inner `colorizeImageData()` function. The inner is injectable, to allow the actual coloring logic to be handled externally. 
  In my own use, it's handled by a webworker.

- new config option `config.colorizeImageData`, with default setting to match upstream's drawing logic.
  ```javascript
    config.colorizeImageData = function(imageData, palette, opacity, premultiplyAlpha, callback){};
  ```

  `callback` signature:
  ```javascript
    function(modifiedImageData){};
  ```

- `drawAlpha()` has been split into an outer (still called `drawAlpha()`) and inner `drawHeatpath()` function. The inner is injectable, to allow the shape
  of each heatpoint to be dictated externally.

- new config option `config.drawHeatpath`, with default setting to match upstream's drawing logic.
  ```javascript
    config.drawHeatpath = function(ctx, x, y, radius){};
  ```
  No call to `ctx.beginPath()` or `ctx.closePath()` should be made, they're handled by the caller, and you will incur performance penalties if you ignore this. Be sure to use `moveTo()`.

- Some additional debug is output regarding timings.


Upstream's readme is replicated below:


# heatmap.js
heatmap.js is a JavaScript library that can be used to generate web heatmaps with the html5canvas element based on your data.

## How it works
Heatmap instances contain a store in order to colorize the heatmap based on relative data, which means if you're adding only a single datapoint to the store it will be displayed as the hottest(red) spot, then adding another point with a higher count, it will dynamically recalculate. 
The heatmaps are fully customizable - you're welcome to choose your own color gradient, change its opacity, datapoint radius and many more. 

## How to use it
Just add heatmap.js to your webpage and it will create one global object called **heatmapFactory** which you also can access as **h337**.
This global object has a function **create** that takes one argument **config** (Object) and returns a heatmap instance. 
At the configuration object you can specify the following properties in order to customize your heatmap instance:  

- **radius** (optional) Number. That's the radius of a single datapoint in the heatmap** (measured in pixels). Default is 40
- **element** (required) String|HTMLelement. Either provide an element's id or the element itself which should contain the heatmap.
- **visible** (optional) Boolean. Whether the heatmap is visible or not. Default is true
- **gradient** (optional) Object. An object which contains colorstops from 0 to 1. Default is the standard heatmap gradient.
- **opacity** (optional) Number [0-100]. Opacity of the heatmap measured in percent.

Here is an example instanciation:

```javascript
var config = {
    "radius": 30,
    "element": "heatmapEl",
    "visible": true,
    "opacity": 40,
    "gradient": { 0.45: "rgb(0,0,255)", 0.55: "rgb(0,255,255)", 0.65: "rgb(0,255,0)", 0.95: "yellow", 1.0: "rgb(255,0,0)" }
};

var heatmap = heatmapFactory.create(config);
```

After creating the heatmap object you can set a dataset (import), add single datapoints and export the datapoints:

```javascript
// set a dataset
heatmap.store.setDataSet({ max: 10, data: [{x: 10, y: 20, count: 5}, ...]});

// add a single datapoint
heatmap.store.addDataPoint(10, 20);

// export the dataset
var dataSet = heatmap.store.exportDataSet();
```

As you can see a heatmap instance contains a store which stores its datapoints. 
A store has the following functions:  

- **setDataSet(Object)** void. This initializes the heatmap with a dataset. The dataset object has to have the following structure: {max: <maximum count>, data:[{x: <dataPointX>, y: <dataPointY>, count: <valueAtXY>},...]}
- **addDataPoint(Number, Number, [Number])** void. Adds a single datapoint to the store. First parameter is x, second parameter is y. Third parameter is the value, if not specified 1 will be used.
- **exportdataSet()** Object. Returns the store's data as an object with the same structure that the import object at setDataSet has.

## License
heatmap.js is dual-licensed under the MIT and the Beerware license, feel free to use it in your projects. 

## Questions?
Feel free to contact me:  
on my website [patrick-wied.at](http://www.patrick-wied.at "")  
via twitter [@patrickwied](http://twitter.com/#!/patrickwied "")  
or email [contact@patrick-wied.at](mailto:contact@patrick-wied.at "")

