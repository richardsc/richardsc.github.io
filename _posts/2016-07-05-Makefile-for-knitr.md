---
layout:  post
title: "A Makefile for knitr documents"
published:  true
author: "Clark Richards"
date: 2016-07-05
categories: [R, knitr, make]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

One of the best things I've found about using [R](http://r-project.org) for all my scientific work is powerful and easy to use facilities for generating dynamic reports, particularly using the [knitr](http://yihui.name/knitr/) package. The seamless integration of text, code, and the resulting figures (or tables) is a major step toward fully-reproducible research, and I've even found that it's a great way of doing "exploratory" work that allows me to keep my own notes and code contained in the same document.

Being a fan of a "Makefile" approach to working with R scripts, as well as an Emacs/ESS addict, I find the easiest way to automatically run/compile my knitr latex documents is with a Makefile. Below is a template I adapted from [here](https://gist.github.com/halpo/1405945):

    all: pdf
    
    MAINFILE  := **PUT MAIN FILENAME HERE**
    RNWFILES  := 
    RFILES    := 
    TEXFILES  := 
    CACHEDIR  := cache
    FIGUREDIR := figures
    LATEXMK_FLAGS := 
    ##### Explicit Dependencies #####
    ################################################################################
    RNWTEX = $(RNWFILES:.Rnw=.tex)
    ROUTFILES = $(RFILES:.R=.Rout)
    RDAFILES= $(RFILES:.R=.rda)
    MAINTEX = $(MAINFILE:=.tex)
    MAINPDF = $(MAINFILE:=.pdf)
    ALLTEX = $(MAINTEX) $(RNWTEX) $(TEXFILES)
    
    # Dependencies
    $(RNWTEX): $(RDAFILES)
    $(MAINTEX): $(RNWTEX) $(TEXFILES)
    $(MAINPDF): $(MAINTEX) $(ALLTEX) 
    
    .PHONY: pdf tex clean 
    
    pdf: $(MAINPDF)
    
    tex: $(RDAFILES) $(ALLTEX) 
    
    %.tex:%.Rnw
    	Rscript \
    	  -e "library(knitr)" \
    	  -e "knitr::opts_chunk[['set']](fig.path='$(FIGUREDIR)/$*-')" \
    	  -e "knitr::opts_chunk[['set']](cache.path='$(CACHEDIR)/$*-')" \
    	  -e "knitr::knit('$<','$@')"
    
    %.R:%.Rnw
    	Rscript -e "Sweave('$^', driver=Rtangle())"
    
    %.Rout:%.R
    	R CMD BATCH "$^" "$@"
    
    %.pdf: %.tex 
    	latexmk -pdf $<
    
    clean:
    	-latexmk -c -quiet $(MAINFILE).tex
    	-rm -f $(MAINTEX) $(RNWTEX)
    	-rm -rf $(FIGUREDIR)
    	-rm *tikzDictionary
    	-rm $(MAINPDF)
    
