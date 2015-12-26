---
layout: post
title: Using the scan() function in R to read weirdly formatted data
author: "Clark Richards"
date: 2015-09-02
---

I am writing some code to parse a weird data format, and using scan() to suck in everything first. Basically, it’s csv-style lines, but some lines have a different number of fields and are for different things — imaging CTD data interspersed with system messages, where the line is identified by the very first field. Something like:

```
GPS,20150727T120000,-10.1,12.2
MESSAGE, 20150727T120005,Begin descent
CTD,20150727120100,1,25,35
CTD,20150727120200,10,20,34
CTD,20150727120400,100,10,33
MESSAGE,20150727T121000,Begin ascent
CTD,20150727121500,100,10,33
CTD,20150727121600,90,12,33.5
etc ...
```

Anyway, when I was just reading in the CTD fields, everything was fine, but when I started trying to parse the MESSAGE fields, I found that scan() was doing something unexpected with the spaces in the message field, and producing a char vector like:

```
"GPS,20150727T120000,-10.1,12.2"
"MESSAGE, 20150727T120005,Begin" 
"descent"
"CTD,20150727120100,1,25,35"
...
```

Basically, scan() was treating the space between “Begin” and “descent” as a delimiter (as well as the carriage returns).

Anyway, after much attempting to interpret the man page, and trying different things, I discovered that

```r
scan(con, character(), sep='\n')
```

would suck in the entire line as a character vector, which is what I wanted.
