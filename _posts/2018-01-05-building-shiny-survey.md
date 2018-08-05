------
title: Building a survey input tool in shiny
-----

One of my roles at West London Zone involves overseeing the partner data system - that is, the collection of attendance and engagement data from 20 partners, operating around 60 sessions a week, across 9 schools. A major part of the challenge is the flexibility of our needs-based partner allocation model, whereby students can be allocated or removed from partner sessions as their needs change. This means that there at least weekly changes to the lists of students that partners need to report on, and it is my job to ensure that partners are always receiving and reporting on the most up-to-date list of students.

After reading [this post](https://shiny.rstudio.com/articles/persistent-data-storage.html) from Dean Attali on persistent data storage in shiny Apps, I decided to have a go at building a data input tool in shiny. By levering shiny's reactive framework, my thinking was that I could filter data reactively by date, or by other partner session characteristics (e.g. day of session delivery), to ensure that partners were always seeing the most up-to-date list of students when they went to enter data. Additionally, I thought I could cut down on some of the ineffiencies between reporting on 'attendance' and 'engagement'. If a student has not attended a session, by definition, they won't be 'engaged' in the session, so there is no need for their name to appear twice.

The App starts by loading R packages, the data, and the unique Dropbox token. In this section of the App, I've also defined the fields that we want to draw from the partner data.

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

Next, the UI. This includes some instructions for partners to enter data, as well as a passwordInput and DateInput fields, which will be used in the server section to filter the data. I've also set an action button for partners to subimit the data, and added some hidden 'thank you' text using the `hidden` function from the `Shinyjs` package. 

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

Finally, the server function. This section starts by reactively filtering the underlying partner data, depending on the password input field.

```javascript
  server = function(input, output, session) {
    
  
    #Filter list of students by partner and date into reactive dataset  
    filterdata <- reactive({
      if(input$password=="buckingham") { 
        partner %>%
          filter(Partner=="Buckingham Palace",
                 Date.ended<=input$date)
      }
      else{
        if(input$password=="kensington") {
          partner %>%
            filter(Partner=="Kensington Palace", 
                   (is.na(Date.ended) | Date.ended<=input$date),
                   Filter %in% input$filter1)
        }
      }
    })
   ```
The password 'buckingham' has been assigned to our made-up partner "Buckingham Palace', and the password 'kensington' to our second partner "Kensington Palace". For the purposes of this example, Buckingham Palace runs partner sessions only once a week, so they only have the Date input to filter their dataset, but Kensington Palace runs two sessions per week, with different groups of students. So, I've added a second filter `filter1` so that the Kensington Palace data can be filtered by the session was run on a Monday or a Wednesday, and, later on in the server section, defined a reactive UI that only shows once 'kensington' is added into the partner input.

```javascript
    output$filter <- renderUI({
      if(input$password=="kensington") {
        selectInput("filter1", label="Please select the session", choices=c("Monday session","Wednesday session"))
      }
      else{NULL}
    })
```

Two other `renderUI` functions appear in the shiny server. The first is for the list of students for partners to report attendance on, which will only appear once a partner has correctly entered their password in the `passwordInput` field. 

```javascript
    output$attended <- renderUI({
      if(input$password=="buckingham" | input$password=="kensington") {
        new2 <- filterdata()
        checkboxGroupInput("attended", "Please tick if the student attended the session", choices=new2$Name)
      }
      else{NULL}
    })
```

The second is for partners to report on the number of engaged students per session. The code is written such that only students marked as 'attended' also appear in this list.

```javascript
    output$engaged <- renderUI({
      if(input$password=="buckingham" | input$password=="kensington") {
        new3 <- new()
        checkboxGroupInput("engaged", "Please tick if the student was engaged in the session", choices=c(input$attended))
      }
      else{NULL}
    })
 ```
 
 Finally, the server section includes the functions for aggregating and saving the data, as per Dean's original post.
 
 ```javascript
 
     # Whenever a field is filled, aggregate all form data
    formData <- reactive({
      data <- sapply(fields, function(x) input[[x]])
      data
    })
    
    # When the Submit button is clicked, save the form data, and give a thank you message
    observeEvent(input$submit, {
      saveData(formData())
      shinyjs::show(id="thankyou")
    })
    
    
    saveData <- function(data) {
      data <- t(data)
      # Create a unique file name
      fileName <- sprintf("%s_%s.csv", as.integer(Sys.time()), digest::digest(data))
      write.csv(data, fileName, row.names = TRUE, quote = TRUE)
      token <- readRDS("droptoken.rds")
      drop_acc(dtoken = token)    
      drop_upload(fileName, dest = outputDir, dtoken=token)
    }
  }
)
```
 
 

