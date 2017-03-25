---
layout:  post
title: "Downloading and plotting MODIS Chlorophyll-a data"
published: true
author: "Clark Richards"
date: 2017-03-25
categories: [R, oce, modis, chl, sp, raster]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---



I was recently asked for help with a project that involves correlating occurrences of marine animals found at the surface with satellite measurements of surface chlorophyll. Being a physical oceanographer, I'm not too familiar with the details of such data sets (though I did previously help someone [read in MODIS netcdf with the `oce` package](https://rpubs.com/clarkrichards/40319)), but saw it as a nice opportunity to learn a bit about a different data set, but also to gain some new data processing and plotting skills. 

## MODIS Chla data

The MODIS chlorophyll data are provided by NASA through the [OceanColor WEB](https://oceancolor.gsfc.nasa.gov/) site, which provides various manual ways of downloading binary files (e.g. hdf and netCDF) files. For the present application, which potentially required approximately 400 or so images, this wasn't a very appealing option.

A quick google search turned up two very relevant (and fantastic looking!) packages, the [`spnc` package](https://github.com/BigelowLab/spnc) and the [`obpgcrawler` package](https://github.com/BigelowLab/obpgcrawler) (both authored by Ben Tupper from the [Bigelow Laboratory](https://www.bigelow.org/)). `spnc` provides some simplified methods for dealing with "spatial" datasets and netCDF files, and the `obpgcrawler` provides an interface for programmatically downloading various datasets from the NASA Ocean Biology Processing Group (including MODIS!).

### Installing `spnc` and `obpgcrawler`

As the packages are not on CRAN (yet?), they have to be installed using the `devtools` package (and of course all its dependencies, etc). The `obpgcrawler` package also depends on another non-CRAN package, called [`threddscrawler`](https://github.com/BigelowLab/threddscrawler) which must be installed first. Note that due to an issue with OpenDAP support for the `ncdf4` package on windows, the below only works on Linux or OSX.

To install the packages, do:


{% highlight r %}
library(devtools)
# if you don't have threddscrawler installed
install_github("BigelowLab/threddscrawler")
install_github("BigelowLab/obpgcrawler")
install_github("BigelowLab/spnc")
{% endhighlight %}

There are some great examples provided on the Github pages, from which I built on to accomplish what I needed. The below example is pulled straight from the `obpgcrawler` page, to download a subset of the most recent MODIS data and plot it as a "raster" image (more on that later).


{% highlight r %}
library(obpgcrawler)
library(spnc)
library(raster)
{% endhighlight %}



{% highlight text %}
## Loading required package: sp
{% endhighlight %}



{% highlight r %}
query <- obpg_query(top = 'https://oceandata.sci.gsfc.nasa.gov/opendap/catalog.xml',
   platform = 'MODISA', 
   product = 'L3SMI',
   what = 'most_recent',
   greplargs = list(pattern='8D_CHL_chlor_a_4km', fixed = TRUE))
q <- query[[1]]
chl <- SPNC(q$url)
bb <- c(xmin = -77, xmax = -63, ymin = 35, ymax = 46)
r <- chl$get_raster(what = 'chlor_a', bb = bb)
spplot(log10(r), main=paste('MODIS Chla for', format(chl$TIME, '%Y-%d-%m')))
{% endhighlight %}

![plot of chunk example](/figure/source/2017-03-25-modis-chl-data/example-1.png)

## The animal data

The animal data consists of a data frame containing: a date of observation, a longitude, and a latitude. To mimic the data set, I'll just create a single random point and time in the North Atlantic:


{% highlight r %}
library(latticeExtra) # for the `layer()` function
{% endhighlight %}



{% highlight text %}
## Loading required package: lattice
{% endhighlight %}



{% highlight text %}
## Loading required package: RColorBrewer
{% endhighlight %}



{% highlight r %}
date <- as.Date('2017-02-25')
lat <- 43.783179
lon <- -62.860410
query <- obpg_query(top = 'http://oceandata.sci.gsfc.nasa.gov/opendap/catalog.xml',
                    platform = 'MODISA', 
                    product = 'L3SMI',
                    what = 'within',
                    greplargs = list(pattern='8D_CHL_chlor_a_4km', fixed = TRUE),
                    date_filter=c(date-3, date+4) ## find nearest image within one week
                    )
q <- query[[1]]
bb <- c(lon-10, lon+10, lat-10, lat+10) # define a 10x10 degree box around the point
chl <- SPNC(q$url, bb=bb)
r <- chl$get_raster(what='chlor_a')
p <- spplot(log10(r), main=paste0('Image=', format(chl$TIME)),
            scales=list(draw=TRUE), auto.key=list(title="log10[chl]"))
p <- p + layer(panel.points(lon, lat, pch=19, col=1, cex=2))
print(p)
{% endhighlight %}

![plot of chunk animal-point](/figure/source/2017-03-25-modis-chl-data/animal-point-1.png)

Now, we can extract a chlorophyll value from the location, using the `extract()` function:

{% highlight r %}
## Note the `radius` argument, which takes a radius in meters
surface_chl <- mean(extract(r, cbind(lon, lat), buffer=50000)[[1]], na.rm=TRUE)
print(paste0('The surface chlorophyll at ', lon, ', ', lat, ' is: ',
             format(surface_chl, digits=3), ' mg/m^-3'))
{% endhighlight %}



{% highlight text %}
## [1] "The surface chlorophyll at -62.86041, 43.783179 is: 1.09 mg/m^-3"
{% endhighlight %}

## Things to figure out (`sp` plots, rasters, projections, etc)

The world of "spatial" objects (e.g. through the `sp` package), and things that derive from them, is a new one for me. For example, in the `oce` package, we have developed methods for plotting matrices (e.g. `imagep()`) and geographical data (e.g. `mapPlot()`, `mapImage()`, etc) that differ from the GIS-like approach contained in the world of spatial analyses in R. I have long desired to learn more about this "other" world, and so have taken this opportunity with MODIS data to do so.

### Projected rasters and lon/lat labels

The neat thing about `raster` objects is that they contain the coordinate projection information. For example, the coordinate system for the MODIS data that we downloaded can be seen with:

{% highlight r %}
projection(r)
{% endhighlight %}



{% highlight text %}
## [1] "+proj=longlat +datum=WGS84 +ellps=WGS84 +towgs84=0,0,0"
{% endhighlight %}

For those used to doing projected maps in `oce`, this string should be familiar as a proj4 string, which specifies that the coordinate system is simply "longlat" (i.e. not projected). To change the projection of the raster, we can use the `projectRaster()` function to update it to a new reference, e.g. polar stereographic centred on -20 degrees W:

{% highlight r %}
rp <- projectRaster(r, crs='+proj=sterea +lon_0=-20')
{% endhighlight %}
Now, if we use `spplot()` again, we get a raster that is plotted in a projected coordinate:

{% highlight r %}
spplot(log10(rp))
{% endhighlight %}

![plot of chunk plot-projected](/figure/source/2017-03-25-modis-chl-data/plot-projected-1.png)

There are some things I haven't figured out yet, particularly how to plot nice *graticules* (e.g. grid lines for constant latitude and longitude), since if the above is plotted as before with the `scales=` argument, what is plotted are the **projected** coordinates:

{% highlight r %}
spplot(log10(rp), scales=list(draw=TRUE))
{% endhighlight %}

![plot of chunk scales](/figure/source/2017-03-25-modis-chl-data/scales-1.png)

It looks like the [`graticules` package](https://cran.r-project.org/web/packages/graticule/vignettes/graticule.html) will be helpful for this, but it still doesn't appear to be non-trivial. See also [here](https://edzer.github.io/sp/#graticules) for some other good-looking examples.

### Extract a matrix from raster to use `imagep()`

One solution (at least for making maps), would be to extract the matrix data from the raster along with the longitude and latitude vectors. This would then allow for plotting in a projection using `mapImage()` from the `oce` package as I'm used to. Let's try and pull stuff out of the raster object `r`:

{% highlight r %}
lon <- unique(coordinates(r)[,1])
lat <- unique(coordinates(r)[,2])
chl_mat <- as.matrix(r)
dim(chl_mat)
{% endhighlight %}



{% highlight text %}
## [1] 231361      1
{% endhighlight %}
Note that using `as.matrix()` on a raster should extract the matrix values of the object, however in this case it extracts a *vector* with all the values. So, we need to reshape it:

{% highlight r %}
chl_mat <- array(as.matrix(r)[,1], dim=c(length(lon), length(lat)))
imagep(lon, lat, log10(chl_mat), col=oceColorsChlorophyll)
{% endhighlight %}

![plot of chunk reshape](/figure/source/2017-03-25-modis-chl-data/reshape-1.png)
Ok! Now we're getting somewhere! Let's try a projected version:

{% highlight r %}
library(ocedata)
{% endhighlight %}



{% highlight text %}
## Loading required package: testthat
{% endhighlight %}



{% highlight r %}
data(coastlineWorldFine)
mapPlot(coastlineWorldFine, projection=projection(rp),
        longitudelim=range(lon), latitudelim=range(lat))
mapImage(lon, lat, log10(chl_mat), col=oceColorsChlorophyll)
{% endhighlight %}

![plot of chunk mapPlot](/figure/source/2017-03-25-modis-chl-data/mapPlot-1.png)
Awesome!
