---
layout:  post
title: "Back to blogging"
published:  true
author: "Clark Richards"
date: 2015-12-27
categories: [R]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

I've always intended to keep a better blog, partly for the online presence, and partly for the record of things I learn as I muddle my way through computer work and scientific research. My first real attempt, at [Coded Ocean](http://codedocean.wordpress.com), was a bit of a failure -- partly for the normal reasons (lazy, etc), but I believe also because ultimately the WordPress format didn't suit my style very well. Too much moving a mouse and clicking, manually uploading figures and images, and a frustratingly buggy Markdown renderer that often required me to write raw html in my posts.

Since learning about [Jekyll](http://jekyllrb.com), particularly since my friend and colleague (and former supervisor) [Dan Kelley](http://dankelley.github.io) set up his own Jekyll blog, it seems like it's much more along the lines of the kind of blog that would work for me. Specifically, Jekyll:

* lets me host using [GitHub pages](http://pages.github.com)

* lets me write posts in either [Markdown](https://daringfireball.net/projects/markdown/) or [Rmarkdown](http://rmarkdown.rstudio.com/) format

* lets me write said posts in the text editor of my choice, Emacs (specifically [Emacs Mac port](https://github.com/railwaycat/homebrew-emacsmacport/releases))

* lets me write [dynamic document](http://polardispatches.org/open-software-open-science-reproducible-results/)-style posts wherein I can mingle text, code, and results

## Blog details

The guts of this website are based off of the [knitr-hyde](http://statistics.rainandrhino.org/knitr-hyde/) sample blog, which is a great starting point for a R/knitr website and blog, which is in turn built off of the great [poole/hyde](https://github.com/poole/hyde) template. The [source code](https://github.com/richardsc/richardsc.github.io) is hosted on GitHub (of course).
