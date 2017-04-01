---
layout:  post
title: "Adding NOAA bottom profile to section plots"
published: true
author: "Clark Richards"
date: 2017-04-01
categories: [R, oce, section, marmap]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

I use the `section-class` plotting method in the [`oce` package](http://dankelley.github.io/oce) *a lot*. It's one of the examples I really like showing to new oceanographic users of R and `oce`, to see the power in making quick plots from potentially very complicated data sets. A canonical example is to use the built-in `data(section)` dataset:


{% highlight r %}
library(oce)
data(section)
plot(section, which='temperature')
{% endhighlight %}

![plot of chunk example](/figure/source/2017-04-01-bottom-profiles-on-section-plots/example-1.png)

Note the grey bottom profile that is automatically overlaid on the plot -- the values for those points come from the individual stations in the `section` object, from the `waterDepth` metadata item in each of the stations in the section. The values can be extracted to a vector with our trusty friend `lapply`[^1]:


{% highlight r %}
depth <- unlist(lapply(section[['station']], function(x) x[['waterDepth']]))
distance <- unique(section[['distance']])
plot(distance, depth, type='l')
{% endhighlight %}

![plot of chunk depth](/figure/source/2017-04-01-bottom-profiles-on-section-plots/depth-1.png)

However, many CTD datasets don't automatically include the water depth at the station, and even if they do the large spacing between stations may make the bottom look clunky. 

## Using the `marmap` package to add a high res bottom profile

To add a nicer looking profile to the bottom, we can take advantage of the awesome [`marmap` package](https://cran.r-project.org/web/packages/marmap/index.html), which can download bathymetric data from NOAA. 

To add a nice looking bottom profile to our section plot, we can use the `getNOAA.bathy()` and `get.depth()` functions. Note the `resolution=1` argument, which downloads the highest resolution data available from NOAA (1 minute resolution), and the `keep=TRUE` argument, which saves a local copy of the data to prevent re-downloading every time the script is re-run (note that at 1 minute resolution the csv file obtained below is 29 MB):


{% highlight r %}
library(marmap)
{% endhighlight %}



{% highlight text %}
## 
## Attaching package: 'marmap'
{% endhighlight %}



{% highlight text %}
## The following object is masked from 'package:oce':
## 
##     plotProfile
{% endhighlight %}



{% highlight text %}
## The following object is masked from 'package:grDevices':
## 
##     as.raster
{% endhighlight %}



{% highlight r %}
lon <- section[["longitude", "byStation"]]
lat <- section[["latitude", "byStation"]]
lon1 <- min(lon) - 0.5
lon2 <- max(lon) + 0.5
lat1 <- min(lat) - 0.5
lat2 <- max(lat) + 0.5

## get the bathy matrix -- 29 MB file
b <- getNOAA.bathy(lon1, lon2, lat1, lat2, resolution=1, keep=TRUE)
{% endhighlight %}



{% highlight text %}
## File already exists ; loading 'marmap_coord_-74.1727;35.703;-8.0263;38.7373_res_1.csv'
{% endhighlight %}



{% highlight r %}
plot(section, which="temperature", showBottom=FALSE)
loni <- seq(min(lon), max(lon), 0.1)
lati <- approx(lon, lat, loni, rule=2)$y
dist <- rev(geodDist(loni, lati, alongPath=TRUE))
bottom <- get.depth(b, x=loni, y=lati, locator=FALSE)
polygon(c(dist, min(dist), max(dist)), c(-bottom$depth, 10000, 10000), col='grey')
{% endhighlight %}

![plot of chunk marmap-example](/figure/source/2017-04-01-bottom-profiles-on-section-plots/marmap-example-1.png)

Nice!

[^1]: See also [http://clarkrichards.org/r/2015/09/01/using-the-r-apply-family/](Using the R apply family with oce objects)
