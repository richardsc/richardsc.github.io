---
layout:  post
title: "Using the R apply() family with oce objects"
published:  true
author: "Clark Richards"
date: 2015-09-01
categories: [R]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

# Introduction

In the [oce](http://dankelley.github.io/oce) package, the various different data formats are stored in consistently structured objects. In this post, I’ll explore a way to access elements of multiple oce objects using the R lapply(), from the apply [family of functions](https://stat.ethz.ch/R-manual/R-devel/library/base/html/lapply.html).

# Example with a `ctd` object

The objects always contain three fields (or “slots”): `metadata`, `data`, and `processingLog`. The layout of the object can be visualized using the `str()` command, like:

```r
library(oce)
```

```
## Loading required package: methods
## Loading required package: gsw
```

```r
data(ctd)
str(ctd)
```

which produces something like:

```r
Formal class 'ctd' [package "oce"] with 3 slots
  ..@ metadata     :List of 26
  .. ..$ header                  : chr [1:42] "* Sea-Bird SBE 25 Data File:"
  .. ..$ type                    : chr "SBE"
  .. ..$ conductivityUnit        : chr "ratio"
  .. ..$ temperatureUnit         : chr "IPTS-68"
  .. ..$ systemUploadTime        : POSIXct[1:1], format: "2003-10-15 11:38:38"
  .. ..$ station                 : chr "Stn 2"
  .. ..$ date                    : POSIXct[1:1], format: "2003-10-15 11:38:38"
  .. ..$ startTime               : POSIXct[1:1], format: "2003-10-15 11:38:38"
  .. ..$ latitude                : num 44.7
  .. ..$ longitude               : num -63.6
  ..@ data         :List of 9
  .. ..$ scan         : int [1:181] 130 131 132 133 134 135 136 137 138 139 ...
  .. ..$ time         : num [1:181] 129 130 131 132 133 134 135 136 137 138 ...
  .. ..$ pressure     : num [1:181] 1.48 1.67 2.05 2.24 2.62 ...
  .. ..$ depth        : num [1:181] 1.47 1.66 2.04 2.23 2.6 ...
  .. ..$ temperature  : num [1:181] 14.2 14.2 14.2 14.2 14.2 ...
  .. ..$ salinity     : num [1:181] 29.9 29.9 29.9 29.9 29.9 ...
  .. ..$ temperature68: num [1:181] 14.2 14.2 14.2 14.2 14.2 ...
  ..@ processingLog:List of 2
  .. ..$ time : POSIXct[1:5], format: "2015-08-18 19:22:36" "2015-08-18 19:22:36" ...
  .. ..$ value: chr [1:5] "create 'ctd' object" "ctdAddColumn(x = res, column = swSigmaTheta(res@data$salinity,     res@data$temperature, res@data$pressure), name = "sigmaThet"| __truncated__ "read.ctd.sbe(file = file, processingLog = processingLog)" "converted temperature from IPTS-69 to ITS-90" ...
```

(where I’ve trimmed a few lines out just to make it shorter).

For a single object, there are several ways to access the information contained in the object. The first (and generally recommended) way is to use the `[[` accessor — for example if you wanted the temperature values from a ctd object you would do

```r
T <- ctd[['temperature']]
```

Another way is to access the element directly, by using the slot and list syntax, like:

```r
T <- ctd@data$temperature
```

The disadvantage to the latter is that it requires knowledge of exactly where the desired field is in the object structure, and is brittle to downstream changes in the oce source.

# Working with multiple objects

Frequently, especially with CTD data, it is common to have to work with a number of individual ctd objects — usually representing different casts. One way of organizing such objects, particularly if they share a common instrument, or ship, or experiment etc, is to collect them into a list.

For example, we could loop through a directory of individual cast files (or extract multiple casts from one file using `ctdFindProfiles()`), and append each one to a list like:


```r
files <- dir(pattern='*.cnv')
casts <- list()
for (ifile in 1:length(files)) {
    casts[[ifile]] <- read.oce(files[ifile])
}
```

If we summarize the new casts list, we can see that it’s filled with ctd objects:

```r
str(casts, 1) # the "1" means just go one level deep
```

# Extracting fields from multiple objects at once

Say we want to extract all the temperature measurements from each object in our new list? How could we do it?

The brute force approach would be to loop through the list elements, and append the temperature field to a vector, maybe something like:

```r
T_all <- NULL
for (i in 1:length(casts)) {
    T_all <- c(T_all, casts[[i]][['temperature']])
}
```

But in R, there’s a more elegant way — `lapply()`!

```r
T_all <- unlist(lapply(casts, function(x) x[['temperature']]))
```
