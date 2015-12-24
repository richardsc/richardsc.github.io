---
layout: post
title: Hello World!
---

This is just a test post for my new github.io Jekyll site based on [poole/hyde](https://github.com/poole/hyde).

## Basic markdown for post content

So I can do neat things like:

* headings
* lists
* links
* ...

## Awesome!

What happens if I do latex?

$$ \alpha = \int_{-\infty}^{+\infty} \, f(x)\, dx $$

Hm. Doesn't seem to work. I wonder if I can get MathJax working ...

## R code, and stuff

Part of what I want to do here is to write [RMarkdown](http://rmarkdown.rstudio.com/) documents, then run them through [knitr](http://yihui.name/knitr/) to make nicely formatted blog posts with code and figures, etc.

I can at least make R code blocks like this:
{% highlight r %}
library(oce)
data(ctd)
theta <- swTheta(ctd)
plot(ctd)
{% endhighlight $}
