# tiles.rdnt.io

[tiles.rdnt.io](http://tiles.rdnt.io) is web map tiling service that lets anyone tap in to the power of 
[Cloud Optimized GeoTIFFs](http://cogeo.org). It is hosted by [Radiant.Earth](http://radiant.earth) on 
[AWS Lambda](https://aws.amazon.com/lambda/), and currently is open for anyone to try out and use in applications.

This repository is the version of [marblecutter-virtual](https://github.com/mojodna/marblecutter-virtual) deployed on 
[tiles.rdnt.io](http://tiles.rdnt.io).

The service is hosted for free, for as long as it is affordable for Radiant. If you are a heavy user of the service
we encourage you to stand up your own marblecutter-virtual, which is quite easy to do.

## Using the service

If you just want to view map data served up with the service then you can make use of [cog-map](http://www.cogeo.org/map). 
You can enter the URL to an online COG or to a VRT that describes a collection of COGs. For example the following will load a collection of COGs from NOAA that were collected following the Nashville, TN tornado:
<pre><code>https://www.cogeo.org/map/#/url/https%3A%2F%2Fnoaa-eri-pds.s3.amazonaws.com%2F2020_Nashville_Tornado%2F20200307a_RGB%2F20200307a_COG.vrt/center/-85.58207,36.177/zoom/20</pre></code>


If you are a developer who wants to use the tiles in their mapping application then read on!

(Image credits CC-BY-SA from [Planet disaster data](https://www.planet.com/disaster/hurricane-harvey-2017-08-28/)

### Endpoints

#### `/bounds` - Source image bounds (in geographic coordinates)

##### Parameters

* `url` - a URL to a valid COG. Required.

##### Example

```bash
$ curl "http://tiles.rdnt.io/bounds?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif"
{
  "bounds": [
    -95.46993599071261,
    28.86905396361014,
    -95.2386152334213,
    29.068190805522605
  ],
  "url": "https://s3-us-west-2.amazonaws.com/planet-disaster-data/hurricane-harvey/SkySat_Freeport_s03_20170831T162740Z3.tif"
}
```

#### `/tiles/{z}/{x}/{y}` - Tiles

##### Parameters

* `url` - a URL to a valid COG. Required.
* `rgb` - Source bands to map to RGB channels. Defaults to `1,2,3`.
* `nodata` - a custom NODATA value.
* `linearStretch` - whether to stretch output to match min/max values present in
  the source. Useful for raw sensor output, e.g. earth observation (EO) data.

`@2x` can be added to the filename (after the `{y}` coordinate) to request
retina tiles. The map preview will detect support for retina displays and
request tiles accordingly.

PNGs or JPEGs will be rendered depending on the presence of NODATA values in the
source image (surfaced as transparency in the output).

##### Examples

```bash
$ curl "http://tiles.rdnt.io/tiles/14/3851/6812@2x?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif" | imgcat
```

![RGB](docs/rgb.png)

```bash
$ curl "http://tiles.rdnt.io/tiles/14/3851/6812@2x?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif&rgb=1,1,1" | imgcat
```

![greyscale](docs/greyscale.png)

```bash
$ curl "http://tiles.rdnt.io/tiles/14/3851/6812@2x?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif&rgb=1,1,1&linearStretch=true" | imgcat
```

![greyscale stretched](docs/greyscale_stretched.png)

#### `/tiles` - TileJSON

##### Parameters

See tile parameters.

##### Example

```bash
$ curl "http://tiles.rdnt.io/tiles?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif"
{
  "bounds": [
    -95.46993599071261,
    28.86905396361014,
    -95.2386152334213,
    29.068190805522605
  ],
  "center": [
    -95.35427561206696,
    28.968622384566373,
    15
  ],
  "maxzoom": 21,
  "minzoom": 8,
  "name": "Untitled",
  "tilejson": "2.1.0",
  "tiles": [
    "//tiles.rdnt.io/tiles/{z}/{x}/{y}?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif"
  ]
}
```

#### `/preview` - Preview

##### Parameters

See tile parameters.

##### Example

`http://tiles.rdnt.io/preview?url=https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fplanet-disaster-data%2Fhurricane-harvey%2FSkySat_Freeport_s03_20170831T162740Z3.tif`

### Using Leaflet or OpenLayers

These tiles should work with Leaflet or OpenLayers quite easily. For code examples see:

[marblecutter preview](https://github.com/mojodna/marblecutter/blob/master/marblecutter/templates/preview.html) (leaflet)

[cog-map](https://github.com/radiantearth/cog-map/blob/master/main.js) (openlayers)
