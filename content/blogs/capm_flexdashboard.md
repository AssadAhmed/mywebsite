---
categories:  
- ""    #the front matter should be like the one found in, e.g., blog2.md. It cannot be like the normal Rmd we used
- ""
date: "2023-06-18"
description: Portfolio Growth Overtime & CAPM Dashboard for selected stock tickers # the title that will show up once someone gets to this page
draft: false
image: spices.jpg # save picture in \static\img\blogs. Acceptable formats= jpg, jpeg, or png . Your iPhone pics wont work

keywords: ""
slug: Portfolio_CAPM # slug is the shorthand URL address... no spaces plz
title: Portfolio Growth Overtime & CAPM Dashboard
  
---

<script src="/rmarkdown-libs/bootstrap-datepicker-js/js/bootstrap-datepicker.min.js"></script>
<script>(function() {
        var datepicker = $.fn.datepicker.noConflict();
        $.fn.bsDatepicker = datepicker;
      })();
     </script>
<link href="/rmarkdown-libs/bootstrap-datepicker-css/css/bootstrap-datepicker3.min.css" rel="stylesheet" />
<script src="/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>
<script src="/rmarkdown-libs/plotly-binding/plotly.js"></script>
<script src="/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>
<script src="/rmarkdown-libs/jquery/jquery.min.js"></script>
<script src="/rmarkdown-libs/proj4js/proj4.js"></script>
<link href="/rmarkdown-libs/highcharts/css/motion.css" rel="stylesheet" />
<script src="/rmarkdown-libs/highcharts/highcharts.js"></script>
<script src="/rmarkdown-libs/highcharts/highcharts-3d.js"></script>
<script src="/rmarkdown-libs/highcharts/highcharts-more.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/stock.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/map.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/data.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/exporting.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/offline-exporting.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/drilldown.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/item-series.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/overlapping-datalabels.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/annotations.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/export-data.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/funnel.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/heatmap.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/treemap.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/sankey.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/dependency-wheel.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/organization.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/solid-gauge.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/streamgraph.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/sunburst.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/vector.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/wordcloud.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/xrange.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/tilemap.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/venn.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/gantt.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/timeline.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/parallel-coordinates.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/bullet.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/coloraxis.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/dumbbell.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/lollipop.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/series-label.js"></script>
<script src="/rmarkdown-libs/highcharts/plugins/motion.js"></script>
<script src="/rmarkdown-libs/highcharts/custom/reset.js"></script>
<script src="/rmarkdown-libs/highcharts/modules/boost.js"></script>
<script src="/rmarkdown-libs/highchart-binding/highchart.js"></script>

``` r
library(tidyverse)
library(shiny)
library(highcharter)
```

    ## Warning: package 'highcharter' was built under R version 4.3.1

``` r
library(tidyquant)
library(timetk)
library(scales)
library(broom)
library(highcharter)
library(plotly)
```

# Sidebar

``` r
fluidRow(
  column(6, # column width
         
  # variable stock 1, show "Stock 1", and choose by default "AAPL"
  textInput("stock1", "Stock 1", "AAPL")),
  
  
  column(5,
  
  # weight of stock 1, show "Weight %", 25% by default, anc check weight is 0-100
  numericInput("w1", "Weight %", 25, min = 0, max = 100))
)  
```

<div class="row">
<div class="col-sm-6">
<div class="form-group shiny-input-container">
<label class="control-label" id="stock1-label" for="stock1">Stock 1</label>
<input id="stock1" type="text" class="form-control" value="AAPL"/>
</div>
</div>
<div class="col-sm-5">
<div class="form-group shiny-input-container">
<label class="control-label" id="w1-label" for="w1">Weight %</label>
<input id="w1" type="number" class="form-control" value="25" min="0" max="100"/>
</div>
</div>
</div>

``` r
fluidRow(
  column(6,
  textInput("stock2", "Stock 2", "BA")),
  column(5,
  numericInput("w2", "Weight %", 25, min = 0, max = 100))
)
```

<div class="row">
<div class="col-sm-6">
<div class="form-group shiny-input-container">
<label class="control-label" id="stock2-label" for="stock2">Stock 2</label>
<input id="stock2" type="text" class="form-control" value="BA"/>
</div>
</div>
<div class="col-sm-5">
<div class="form-group shiny-input-container">
<label class="control-label" id="w2-label" for="w2">Weight %</label>
<input id="w2" type="number" class="form-control" value="25" min="0" max="100"/>
</div>
</div>
</div>

``` r
fluidRow(
  column(6,
  textInput("stock3", "Stock 3", "DIS")),
  column(5,
  numericInput("w3", "Weight %", 20, min = 0, max = 100))
)
```

<div class="row">
<div class="col-sm-6">
<div class="form-group shiny-input-container">
<label class="control-label" id="stock3-label" for="stock3">Stock 3</label>
<input id="stock3" type="text" class="form-control" value="DIS"/>
</div>
</div>
<div class="col-sm-5">
<div class="form-group shiny-input-container">
<label class="control-label" id="w3-label" for="w3">Weight %</label>
<input id="w3" type="number" class="form-control" value="20" min="0" max="100"/>
</div>
</div>
</div>

``` r
fluidRow(
  column(6,
  textInput("stock4", "Stock 4", "GS")),
  column(5,
  numericInput("w4", "Weight %", 20, min = 0, max = 100))
)
```

<div class="row">
<div class="col-sm-6">
<div class="form-group shiny-input-container">
<label class="control-label" id="stock4-label" for="stock4">Stock 4</label>
<input id="stock4" type="text" class="form-control" value="GS"/>
</div>
</div>
<div class="col-sm-5">
<div class="form-group shiny-input-container">
<label class="control-label" id="w4-label" for="w4">Weight %</label>
<input id="w4" type="number" class="form-control" value="20" min="0" max="100"/>
</div>
</div>
</div>

``` r
fluidRow(
  column(6,
  textInput("stock5", "Stock 5", "MRK")),
  column(5,
  numericInput("w5", "Weight %", 10, min = 0, max = 100))
)
```

<div class="row">
<div class="col-sm-6">
<div class="form-group shiny-input-container">
<label class="control-label" id="stock5-label" for="stock5">Stock 5</label>
<input id="stock5" type="text" class="form-control" value="MRK"/>
</div>
</div>
<div class="col-sm-5">
<div class="form-group shiny-input-container">
<label class="control-label" id="w5-label" for="w5">Weight %</label>
<input id="w5" type="number" class="form-control" value="10" min="0" max="100"/>
</div>
</div>
</div>

``` r
fluidRow(
  column(7,
  dateInput("date", "Starting Date", "2007-01-01", format = "yyyy-mm-dd"))
)
```

<div class="row">
<div class="col-sm-7">
<div id="date" class="shiny-date-input form-group shiny-input-container">
<label class="control-label" id="date-label" for="date">Starting Date</label>
<input type="text" class="form-control" aria-labelledby="date-label" title="Date format: yyyy-mm-dd" data-date-language="en" data-date-week-start="0" data-date-format="yyyy-mm-dd" data-date-start-view="month" data-initial-date="2007-01-01" data-date-autoclose="true" data-date-dates-disabled="null" data-date-days-of-week-disabled="null"/>
</div>
</div>
</div>

``` r
actionButton("go", "Submit")
```

<button id="go" type="button" class="btn btn-default action-button">Submit</button>

``` r
myportfolio_data <- eventReactive(input$go, {

# Get symbols from user  
symbols <- c(input$stock1, input$stock2, input$stock3, input$stock4, input$stock5)

# Get weights from user and make sure they add up to 100
weights <- c(input$w1/100, input$w2/100, input$w3/100, input$w4/100, input$w5/100)
validate(need(input$w1 + input$w2+ input$w3 + input$w4+input$w5 == 100,
            "Portfolio weights must sum to 100%!"))


myStocks <- symbols %>% 
  tq_get(get  = "stock.prices",
         from = input$date,
         to   = Sys.Date()) %>%
  group_by(symbol) 

# get prices for SPY, the SP500 ETF
spy <- tq_get("SPY", get  = "stock.prices",
              from = input$date,
              to   =  Sys.Date()) 

#calculate monthly  returns for the chosen stocks
myStocks_returns_monthly <- myStocks %>% 
  tq_transmute(select     = adjusted, 
               mutate_fun = periodReturn, 
               period     = "monthly", 
               type       = "arithmetic",
               col_rename = "monthly_return",
               cols = c(nested.col)) 


#calculate SPY monthly  returns
spy_returns_monthly <- spy %>%
  tq_transmute(select     = adjusted, 
               mutate_fun = periodReturn, 
               period     = "monthly", 
               type       = "arithmetic",
               col_rename = "SPY_return",
               cols = c(nested.col))

#calculate portfolio monthly  returns - weights * returns
portfolio_returns_tq_rebalanced_monthly <-  tq_portfolio(data = myStocks_returns_monthly,
             assets_col = symbol,
             returns_col = monthly_return,
             weights = weights,
             col_rename = "monthly_return",
             wealth.index = FALSE)
  
myportfolio_data <- left_join(portfolio_returns_tq_rebalanced_monthly, 
                              spy_returns_monthly, 
                              by="date") %>% 
                              na.omit() %>% 
                    mutate(
                      # cumsum() and cumprod() calcuale the running sum/product
                      # here we use cumprod, as we calcualte return using type=arithmetic
                      # if we had calculated return using log, we would need cumsum
                        portfolio_growth =  100 * cumprod(1 + monthly_return),
                        sp500_growth = 	100 * cumprod(1 + SPY_return)
                    )
})

portfolio_model_augmented <- eventReactive(input$go, {
  
  myportfolio_data <- myportfolio_data()
  
  
  portfolio_model_augmented <- 
    myportfolio_data %>% 
    lm(monthly_return ~ SPY_return, data = .) %>% 
    augment() %>% 
    mutate(date = myportfolio_data$date)
  
})
```

# Choose 5 stocks and a starting date

## Row

### Growth of \$100 invested in portfolio (blue) vs. SP500 (red)

``` r
#use plotly to create interactive chart, so when we place our cursor on ti, we can see values

renderPlotly({
  
  fubar1 <- myportfolio_data() %>% 
    ggplot(aes(x=date))+
    geom_line(aes(y=portfolio_growth),
              colour="#001e62")+
    geom_line(aes(y=sp500_growth),
              colour="tomato")+
    scale_y_continuous(labels = scales::dollar)+
    theme_minimal()+
    labs(x="", y="")  
    
  ggplotly(fubar1)
})
```

<div class="plotly html-widget html-widget-output shiny-report-size shiny-report-theme html-fill-item-overflow-hidden html-fill-item" id="out1a26c03f44a58724" style="width:100%;height:400px;"></div>

## Row 2

### CAPM: Portfolio Returns vs Market Index (SP500) returns

``` r
#use highchart to get interactive scatter plot

renderHighchart({

myportfolio_data <- myportfolio_data()
portfolio_model_augmented <- portfolio_model_augmented()

highchart() %>% 
  hc_title(text = "Portfolio Returns vs SP500 returns with Regression Line") %>% 
  hc_add_series(portfolio_model_augmented, 
                type = "scatter",
                color = "cornflowerblue",
                hcaes(x = round(SPY_return, 4), 
                      y = round(monthly_return, 4),
                      date = date), 
                name = "Returns") %>%
  hc_add_series(portfolio_model_augmented, 
                 type = "line", 
                 enableMouseTracking = FALSE,
                 hcaes(x = SPY_return, y = .fitted), 
                 name = "CAPM Beta = Slope of Line") %>% 
  hc_xAxis(title = list(text = "Market Returns")) %>% 
  hc_yAxis(title = list(text = "Portfolio Returns")) %>% 
  hc_tooltip(formatter = JS("function(){
     return ('portfolio: ' + this.y + '  SP500: ' + this.x +  
     '  date: ' + this.point.date)}"))%>% 
  hc_add_theme(hc_theme_flat())

})
```

<div class="highchart html-widget html-widget-output shiny-report-size html-fill-item-overflow-hidden html-fill-item" id="out46d587b059b9a360" style="width:100%;height:400px;"></div>

## Row 3

### CAPM Model Results, fitted through entire time interval

``` r
renderTable({

  myportfolio_data <- myportfolio_data()
  
    myportfolio_data %>% 
    lm(monthly_return ~ SPY_return, data = .) %>% 
    tidy(conf.int=TRUE) %>% 
  mutate(term = c("alpha", "beta"))
}, digits = 4)
```

<div id="out0a085a462a6e408d" class="shiny-html-output"></div>
