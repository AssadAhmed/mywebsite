---
categories:  
- ""    #the front matter should be like the one found in, e.g., blog2.md. It cannot be like the normal Rmd we used
- ""
date: "2023-06-18"
description: Mapping # the title that will show up once someone gets to this page
draft: false
image: doorway.jpg # save picture in \static\img\blogs. Acceptable formats= jpg, jpeg, or png . Your iPhone pics wont work

keywords: ""
slug: flexdashboard # slug is the shorthand URL address... no spaces plz
title: Flexdashboard with Gapminder
runtime: shiny  
---

``` r
library(flexdashboard)
```

    ## Warning: package 'flexdashboard' was built under R version 4.3.1

``` r
library(shiny)
library(ggplot2)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(gapminder)

# use bigger font size
theme_set(theme_bw(base_size = 16)) 

# Define UI
ui <- fluidPage(
  
  # Sidebar layout with input controls
  sidebarLayout(
    
    # Sidebar panel with input controls
    sidebarPanel(
      selectInput("country", 
                  "Select one or more countries:", 
                  choices = unique(gapminder$country), 
                  multiple = TRUE),      
      sliderInput("years", 
                  "Select a range of years:", 
                  min = 1952L, max = 2007L, 
                  value = as.integer(c(1952, 2007)), 
                  step = 5)
    ),
    
    # Main panel with output plots
    mainPanel(
      plotOutput("life_expectancy_plot"),
      plotOutput("gdp_plot"),
      plotOutput("population_plot")
    )
  )
)
# Define server
server <- function(input, output) {
  
  # Create reactive subset of gapminder data based on user input
  selected_data <- reactive({
    gapminder %>% filter(country %in% input$country,
                         year >= input$years[1], 
                         year <= input$years[2])
  })
  
  # Render life expectancy plot
  output$life_expectancy_plot <- renderPlot({
    ggplot(selected_data(), aes(x = year, y = lifeExp, group = country, colour = country)) +
      geom_point()+
      geom_line() +
      ggtitle("Life Expectancy Over Time") +
      xlab("Year") +
      ylab("Life Expectancy") 
  })
  
  # Render GDP plot
  output$gdp_plot <- renderPlot({
    ggplot(selected_data(), aes(x = year, y = gdpPercap,group = country, colour = country)) +
      geom_point()+
      geom_line() +
      ggtitle("GDP per Capita Over Time") +
      xlab("Year") +
      ylab("GDP per Capita") 
  })
  
  # Render population plot
  output$population_plot <- renderPlot({
    ggplot(selected_data(), aes(x = year, y = pop, group = country, colour = country)) +
      geom_point()+
      geom_line() +
      ggtitle("Population Over Time") +
      xlab("Year") +
      ylab("Population") 
  })
}


#Run the application
shinyApp(ui = ui, server = server)
```

    ## PhantomJS not found. You can install it with webshot::install_phantomjs(). If it is installed, please make sure the phantomjs executable can be found via the PATH variable.

<div style="width: 100% ; height: 400px ; text-align: center; box-sizing: border-box; -moz-box-sizing: border-box; -webkit-box-sizing: border-box;" class="muted well">Shiny applications not supported in static R Markdown documents</div>
