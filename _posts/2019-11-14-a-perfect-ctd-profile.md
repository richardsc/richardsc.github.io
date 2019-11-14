---
layout:  post
title: "A perfect CTD profile"
published: true
author: "Clark Richards"
date: 2019-11-14
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

I love the $$\tanh$$ function. A lot. It's such a perfect model for a
density interface in the ocean, that it is commonly used in
theoretical and numerical models and I regularly used it for both
research and demonstration/example purposes. Behold, a $$\tanh$$ interface:

$$ T(z) = T_0 + \delta T \tanh \left( \frac{z-z_0}{dz} \right) $$


{% highlight r %}
T <- function(z, T0=10, dT=5, z0=-25, dz=5) T0 + dT*tanh((z - z0)/dz)
z <- seq(0, -60)
plot(T(z), z)
{% endhighlight %}

![plot of chunk unnamed-chunk-1](/figure/source/2019-11-14-a-perfect-ctd-profile/unnamed-chunk-1-1.png)

But whenever I use it, especially for teaching, I'm always saying how it's idealized and really doesn't represent what an ocean interface actually looks like. UNTIL NOW.


{% highlight r %}
library(oce)
ctd <- read.oce('../assets/D19002034.ODF')
{% endhighlight %}



{% highlight text %}
## Warning in read.odf(file = file, columns = columns, debug = debug -
## 1): "CRAT_01" should be unitless, but the file states the unit as "S/
## m" so that is retained in the object metadata. This will likely cause
## problems. See ?read.odf for an example of rectifying this unit error.
{% endhighlight %}



{% highlight r %}
par(mfrow=c(1, 2))
plot(ctd, which=1, type='l')
plot(ctd, which=5)
{% endhighlight %}

![plot of chunk unnamed-chunk-2](/figure/source/2019-11-14-a-perfect-ctd-profile/unnamed-chunk-2-1.png)

Yes, this is real data.

Just how close to a $$\tanh$$ is it?[^dan]

{% highlight r %}
z <- -ctd[["depth"]]
T <- ctd[["temperature"]]
m <- nls(T~a+b*tanh((z-z0)/dz), start=list(a=3, b=1, z0=-10, dz=5))
m
{% endhighlight %}



{% highlight text %}
## Nonlinear regression model
##   model: T ~ a + b * tanh((z - z0)/dz)
##    data: parent.frame()
##        a        b       z0       dz 
##   3.7251   0.9516 -25.0448  -2.7648 
##  residual sum-of-squares: 0.05222
## 
## Number of iterations to convergence: 16 
## Achieved convergence tolerance: 4.394e-06
{% endhighlight %}



{% highlight r %}
plot((T-predict(m))/T * 100, z, type="o", xlab='Percent error')
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/figure/source/2019-11-14-a-perfect-ctd-profile/unnamed-chunk-3-1.png)

[^dan]: My PhD advisor, who also taught me introductory physical oceanography, once said to a class of students while using tanh to describe an idealized interface: "tanh -- it's like 'lunch', only better!"
