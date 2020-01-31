---
title: "MOMA"
author: "VP Nagraj"
date: "February 29, 2016"
output: html_document
runtime: shiny
---

The Museum of Modern Art (MOMA) collection database is publicly available via Github: 

[https://github.com/MuseumofModernArt/collection](https://github.com/MuseumofModernArt/collection)

The data are updated monthly and posted to the repository above in both JSON and CSV formats.

The document below explores this data. It includes a summary, as well as an interactive historgram of the number of acquisitions per department by selected year.

## Load Packages

The code below loads the packages necessary to read in the data, make necessary transformations for analysis and create the interactive plot.

```{r, warning=FALSE, message=FALSE}
library(shiny)
library(dplyr)
library(readr)
library(lubridate)
library(ggplot2)
```


## Load Data

The data are read in using the `read_csv()` function from the **readr** package. And the year acquired variable is derived from the date field using the the `year()` function from the **lubridate** package.

```{r, warning = FALSE, message = FALSE}
moma <- read_csv("https://raw.githubusercontent.com/MuseumofModernArt/collection/master/Artworks.csv")
moma$YearAcquired <- year(moma$DateAcquired)
```

## Summary

```{r, warning = FALSE, message = FALSE}
summary(moma)
```


## Interactive Histogram

The visualization below is an embedded Shiny app. Toggling the *year* select input will update the histogram to show the number of items acquired by department in that given year.

```{r,  warning = FALSE, message = FALSE}
ui <- fluidPage(
    titlePanel(title = "Museum of Modern Art (MOMA) Acquisition Histogram"),
    selectInput("year", "Year", selected = 1999, choices = sort(unique(moma$YearAcquired))),
    plotOutput("barplot")
)
server <- function(input, output) {
    
    output$barplot <-  renderPlot({
        
        ggplot(filter(moma, YearAcquired == input$year), aes(Department)) +
            geom_bar(stat="count", fill = "firebrick") +
            ggtitle("Number of Acquisistions By Department")
        
    })
    
}
shinyApp(ui = ui, server = server)
```

Click here for a Gist of this document.
