---
layout:  post
title: "An R function to shift vectors by a specified lag"
published: true
author: "Clark Richards"
date: 2016-02-09
categories: [R, timeseries]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

Quite of bit of my work involves looking at "shifts" between two time series. There are lots of reasons why shifts are interesting, including such things as:

* phase differences in the tides at two different locations,

* physical separation between sensors on a profiling instrument, and

* clock drifts between two logging sensors.

To accomplish the actual *shifting* of the vectors (I'm not going to discuss here how to determine the amount by which the series should be shifted, since that depends on the parameters of the problem), I created the following function:


{% highlight r %}
shift <- function(x, lag) {
    n <- length(x)
    xnew <- rep(NA, n)
    if (lag < 0) {
        xnew[1:(n-abs(lag))] <- x[(abs(lag)+1):n]
    } else if (lag > 0) {
        xnew[(lag+1):n] <- x[1:(n-lag)]
    } else {
        xnew <- x
    }
    return(xnew)
}
{% endhighlight %}

The function takes as input the vector `x` and a specified integer `lag`, and shifts the input series by that amount. `NA`'s are added to either the beginning or the end (depending on the sign of `lag`) to pad the shifted vector to be the same length as the input. Note that `lag` is defined so that a positive `lag` shifts `x` "to the right", i.e. moves values to a higher index.
