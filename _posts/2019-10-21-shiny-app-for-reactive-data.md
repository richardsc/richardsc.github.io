---
layout:  post
title: "A Shiny app that uses 'reactive' data"
published: true
author: "Clark Richards"
date: 2019-10-21
output:
  html_document:
    mathjax:  default
    fig_caption:  true
---

## Introduction

A powerful, and increasingly useful, tool available to users of R, is the interactive app package known as [Shiny](https://shiny.rstudio.com). Shiny provides a framework for building interactive web apps that can be used for visualization, exploratory data analysis, data quality control -- really anything that could benefit from an interactive environment.

This post will not go into the basics of Shiny. There are many resources on the web for getting started, and of particular value is the [RStudio Shiny gallery](https://shiny.rstudio.com/gallery/).

The purpose of this post is two-fold: to test the embedding of a Shiny app within a jekyll blog post, and to document a test app that I made that uses "reactive data".

## Using Shiny apps with jekyll

In order to run a Shiny app, a "shiny server" is required. The easiest way to do this is to install the necessary R packages (e.g. `install.packages('shiny')`) on your computer, and run a "local" server. This is very easy to do within RStudio, but it can also be done from the system command line, e.g. using a command something like:

```
$ Rscript -e "library(shiny); runApp(port=4321)"
```

Then, opening a web browser to the url 127.0.0.1:4321 will display and run the app. However, the app is only available on the machine that it is being run, and not as a distributed web app that other users can connect to.

For webs that are hosted and made available to others, a proper [Shiny server](https://rstudio.com/products/shiny/shiny-server/) environment must be installed on a properly configured web server. Thankfully Shiny Server is free (and open source), however the configuration of a proper server is not a task for the uninitiated. 

Another option is to use the [ShinyApps.io](https://www.shinyapps.io) service that is provided by RStudio. The free option allows for 5 applications, with a limited amount of server time available (25 hours per month at the time of writing). 

For a Github-pages hosted site (like this one), the lack of an R environment and a shiny server on Github means that any apps to be included in posts must be hosted somewhere else. For this example, I will use my shinyapps.io account.

## Reactive data

The premise of "reactive data" (actually, the concept of "reactive" in the context of Shiny apps is really a fundamental principle) is one where the data being used in the app may change from time to time. In a regular R script, loading data using commands like `load()`, `read.csv()`, etc is easy and will occur every time the script is run. However, in a shiny app, using such commands may result in the data being loaded *only* when the server is started, but then not updating with time as the data being loaded is changed (e.g. if more data is downloaded).

To get around this, we use the shiny function `reactiveFileReader()`, which allows the data reading to occur as a "reactive" object, meaning that it can be updated (at prescribed intervals) even while the app is running.

The example code below is a simple app I made to demonstrate the principle. In it, the user clicks the action button to generate new data, which is saved as a `.rds` object in the server file system. That `rds` object is then read by the file reader, which checks for changes every 1000 milliseconds (1 second) and only re-reads the file if the data has changed when it checks. It's a silly way to make an interactive plot that updates with new data, but demonstrates the principle of reactive data well.


{% highlight r %}
library(shiny)

new_data <- function() {
    d <- list(a=1:100, b=rnorm(100))
    saveRDS(file='data.rds', d)
}
new_data()

server <- function(input, output, session) {
    data <- reactiveFileReader(1000,
                               session,
                               'data.rds',
                               readRDS)

    observeEvent(input$newdata, {
        new_data()
    })
    
    output$plot1 <- renderPlot({
        hist(data()[['b']])
    })
}

ui <- fluidPage(    
    
    ## Give the page a title
    titlePanel("Title"),
    
    ## Generate a row with a sidebar
    sidebarLayout(      
        
        ## Define the sidebar with one input
        sidebarPanel(
            actionButton(inputId = 'newdata',
                         label = 'Generate new data')
        ),
        
        ## Create a spot for the barplot
        mainPanel(
            plotOutput("plot1")  
        )
        
    )
)

shinyApp(ui = ui, server = server)
{% endhighlight %}

## The app

To include the app in the post, we upload it to shinyapps.io, and then include it as an `iframe` object, with (note the html comment tags around the `iframe` code chunk to keep it from evaluating in HTML):

```
<!-- <iframe id="example1" src="https://clarkrichards.shinyapps.io/reactiveData/" 
style="border: non; width: 100%; height: 500px" 
frameborder="0">
</iframe> -->
```

<br>

<iframe id="example1" src="https://clarkrichards.shinyapps.io/reactiveData/"
style="border: non; width: 100%; height: 500px"
frameborder="0">
</iframe>

<br>
