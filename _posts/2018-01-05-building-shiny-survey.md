------
title: Building a survey input tool in shiny
-----

One of my roles at West London Zone involves overseeing the partner data system - that is, the collection of attendance and engagement data from 20 partners, operating around 60 sessions a week, across 9 schools. A major part of the challenge is the flexibility of our needs-based partner allocation model, whereby students can be allocated or removed from partner sessions as their needs change. This means that there at least weekly changes to the lists of students that partners need to report on, and it is my job to ensure that partners are always receiving and reporting on the most up-to-date list of students.

After reading [this post](https://shiny.rstudio.com/articles/persistent-data-storage.html) from Dean Attali on persistent data storage in shiny Apps, I decided to have a go at building a data input tool in shiny. By levering shiny's reactive framework, my thinking was that I could filter data reactively by date, or by other partner session characteristics (e.g. day of session delivery), to ensure that partners were always seeing the most up-to-date list of students when they went to enter data. Additionally, I thought I could cut down on some of the ineffiencies between reporting on 'attendance' and 'engagement'. If a student has not attended a session, by definition, they won't be 'engaged' in the session, so there is no need for their name to appear twice.

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

Next, the UI. 

```javascript
shinyApp(
  ui = bootstrapPage(theme=shinytheme("flatly"),
                     navbarPage("Partner Data Portal",
                                tabPanel("Data entry",
                                         sidebarLayout(   
                                           sidebarPanel(
                                             p("Welcome to the Partner Zone Data Portal!"),
                                             p(""),
                                             p("This is where you can enter attendance and engagement data about a session."),
                                             p("Please enter your password and the session date to reveal students' names."),
                                             passwordInput("password", "Please enter your password", ""),
                                             dateInput("date", "What was the date of the session?"),
                                             uiOutput("filter"),
                                             p("Need help? Have any questions?"),
                                             p("Please contact Bridget Suthersan, Senior Data Analyst, on xxx@gmail.com"),
                                             actionButton("submit", "Submit Data"),
                                             p(""),
                                             p(""),
                                             useShinyjs(),
                                             shinyjs::hidden(
                                               div(id = "thankyou", "Thank you, your data has been received!"))),
                                           mainPanel(
                                             img(src='Logo.jpg', align = "right", width=250, height=250),
                                             div(id="myapp",
                                                 uiOutput("attended"),
                                                 uiOutput("engaged"))))),
                                tabPanel("Help and FAQ",
                                         sidebarLayout(   
                                           sidebarPanel(
                                             p("Use this part of the data portal for help or to browse frequently asked questions"),
                                             p(""),
                                             p(strong("How does the data portal work?")),
                                             p(""),
                                             p(strong("I've made a mistake - what do I do?")),
                                             p(""),
                                             p(strong("I've forgotten my password - what do I do?"))),
                                           mainPanel( 
                                             img(src='Logo.jpg', align = "right", width=250, height=250),
                                             p(strong("How does the data portal work?")),
                                             p(""),
                                             p("The data portal is where you can enter attendance and engagement data about our students. We use this portal to collect data for all our students from partners."),
                                             p(strong("I've made a mistake in my data entry! What do I do?")),
                                             p(""),
                                             p("That's okay! Just re-enter your data, and drop Bridget a line at xxx@gmail.com, letting her know what to look out for."),
                                             p(""),
                                             p(strong("I've forgotten my password - what do I do?")),
                                             p(""),
                                             p("You can request a password reset by emailing Bridget Suthersan on xxx@gmail.com"))
                                         ))
                     ))
```
