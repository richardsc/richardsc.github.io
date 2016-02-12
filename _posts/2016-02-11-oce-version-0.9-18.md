---
layout:  post
title: "oce package version 0.9-18 is released!"
published: true
author: "Clark Richards"
date: 2016-02-11
categories: [R, oce]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

Today we released a new version of oce, and it has been uploaded to [CRAN](https://cran.r-project.org/web/packages/oce/). Only the source version is available as of the time of writing, but binary versions for all platforms should become available in the next few days. As always, the best way to install the package is to do:

{% highlight r %}
install.packages("oce")
{% endhighlight %}
at an R prompt. 

Then you can do stuff like:

{% highlight r %}
library(oce)
{% endhighlight %}



{% highlight text %}
## Loading required package: methods
{% endhighlight %}



{% highlight text %}
## Loading required package: gsw
{% endhighlight %}



{% highlight r %}
library(ocedata)
data(levitus)
data(coastlineWorld)
cm <- colormap(levitus$SSS, col=oceColorsSalinity, breaks=seq(30, 37, 0.5))
drawPalette(colormap=cm, drawTriangles = TRUE)
mapPlot(coastlineWorld, projection='+proj=stere +lat_0=90 +lon_0=-60',
        longitudelim=c(-180, 180), latitudelim=c(50, 90), col='grey')
with(levitus, mapImage(longitude, latitude, SSS, colormap=cm, filledContour = TRUE))
mapPolygon(coastlineWorld, col='grey')
mapGrid()
title('Sea Surface Salinity')
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figure/source/2016-02-11-oce-version-0.9-18/unnamed-chunk-2-1.png)

## New features

The previous version of oce was uploaded to CRAN about 9 months ago. In the meantime, we've fixed lots of bugs and added even more improvements. A quick look at the `NEWS` file gives a summary of the enhancements:

    0.9-18
    - improve plot.coastline() and mapPlot()
    - add support for G1SST satellite
    - all objects now have metadata items for units and flags
    - ctdTrim() method renamed: old A and B are new A; old C is new B
    - support more channels and features of rsk files
    - as.adp() added
    - convert argo objects to sections
    - makeSection() deprecated; use as.section() instead
    - read.adp.rdi() handles Teledyne/RDI vsn 23.19 bottom-track data
    - geodXyInverse() added; geod functions now spell out longitude etc
    - read.odf() speeded up by a factor of about 30
    - add colour palettes from Kristen Thyng's cmocean Python package
    - as.oce() added
    - rename 'drifter' class as 'argo' to recognize what it actually handles
    - add oceColorsViridis()
    - interpBarnes() has new argument 'pregrid'
    - binMean2D() has new argument 'flatten'
    - data(topoWorld) now has longitude from -179.5 to 180
    - ODF2oce() added
    - read.odf() handles more data types
    - read.adp.rdi() reads more VmDas (navigational) data
    - ITS-90 is now the default temperature unit
    - ctd objects can have vector longitude and latitude
    - logger class renamed to rsk
    - bremen class added
    - coastlineCut() added
    - rgdal package used instead of local PROJ.4 source code
    - mapproj-style map projections eliminated

Some of the best additions (in my opinion) are:

* addition of a `units` metadata field for objects
* more tools for working with `argo` objects (which have been renamed from `drifter`)
* new color palettes, particularly the [`cmocean` palettes](http://matplotlib.org/cmocean/) from python
* renaming of the `logger` class to `rsk` for data from RBR instruments
* lots of new features for `rsk` objects, including ability to convert to `ctd` objects with `as.ctd()`
* more robust projection handling using the `rgdal` package
* `coastlineCut()` for helping with [UHL (ugly horizontal lines)](https://github.com/dankelley/oce/issues/388) in map projections

## Vignette

The [vignette](https://cran.r-project.org/web/packages/oce/vignettes/oce.html) has also been updated somewhat, using an [Rmarkdown](http://rmarkdown.rstudio.com/) source and with some new examples. 

## Help us out!

You can help! If you find any bugs or have any requests, please open an Issue on the Github development page:

[https://github.com/dankelley/oce/issues](https://github.com/dankelley/oce/issues)

oce has benefited immensely from some great requests recently, and we'd like to keep the momentum going!

## R/oce at AGU Ocean Sciences 2016

If you're going to be at the AGU Ocean Sciences meeting in New Orleans in a couple weeks, make sure you come by to check out the [R/oce tutorial](https://agu.confex.com/agu/os16/meetingapp.cgi/Session/9628) I'll be doing. It's on Wednesday February 24th at 3:00pm in room R03. See you there!
