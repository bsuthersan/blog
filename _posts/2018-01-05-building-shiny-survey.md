------
title: Building a survey input tool in shiny
-----

One of my roles at West London Zone involves overseeing the partner data system - that is, the collection of attendance and engagement data from 20 partners, operating around 60 sessions a week, across 9 schools. A major part of the challenge is the flexibility of our needs-based partner allocation model, whereby students can be allocated or removed from partner sessions as their needs change. This means that there at least weekly changes to the lists of students that partners need to report on, and it is my job to ensure that partners are always receiving and reporting on the most up-to-date list of students.

After reading [this post](https://shiny.rstudio.com/articles/persistent-data-storage.html) from Dean Attali on persistent data storage in shiny Apps, I decided to have a go at building a data input tool in shiny. By levering shiny's reactive framework, my thinking was that I could filter data reactively by date, or by other partner session characteristics (e.g. day of session delivery), to ensure that partners were always seeing the most up-to-date list of students when they went to enter data. Additionally, I thought I could cut down on some of the ineffiencies between reporting on 'attendance' and 'engagement'. If a student has not attended a session, by definition, they won't be 'engaged' in the session. 

The App starts by loading R packages.

```javascript
library(shiny)
library(rdrop2)
library(lubridate)
library(tidyverse)
library(shinyjs)
library(shinythemes)
library(DT)
library(digest)

#Dropbox Token 

token <- readRDS("droptoken.rds")

outputDir <- "responses2"

#Load the data

partner <- read.csv("Data/Data.csv", header = TRUE)

#Format partner data

partner$Date.ended <- as.character(partner$Date.ended)
partner$Date.ended <- as.Date(partner$Date.ended, format="%d/%m/%Y")

# Define the fields we want to save from the form

fields <- c("password", "date","filter", "attended", "engaged")
```

