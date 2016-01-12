---
layout:  post
title: "Turning off Auctex fontification so that columns can align"
published:  true
author: "Clark Richards"
date: 2014-09-02
categories: [Emacs]
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

I love Emacs. I use it for everything, and particularly love it for doing tables in LaTeX because I can easily align everything so that it looks sensible, and rectangle mode makes it easy to move columns around if desired.

That being said, Auctex defaults do some fontification to math-mode super- and subscripts, which cause the horizontal alignments of characters to be off (essentially it is no longer a fixed-width font). To turn this off, do:

```
M-x customize-variable font-latex-fontify-script
```

Beauty!
