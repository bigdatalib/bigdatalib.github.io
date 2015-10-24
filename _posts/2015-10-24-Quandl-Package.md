---
title: "Quandl Data for Energy"
author: "NS"
date: "10/20/2015"
output: html_document
---

## Data Feeds

One of the main challenges in developing data products for energy and commodities trading and risk management is lack of reliable and relevant data that gives you the granularity required to do something useful and at the same time you don't need a week to clean the data. This post examines one of ways you can get the data you need for free. 

## Paid vs. Free Services

The first step in any project is to gather and clean data. Working for a trading shop you usually have access to many paid data feeds. 

In many cases you might not know that you have access, but the rule of thumb is if you are using a data feed provider such as Bloomberg you will have the option to use developer SDK.  In some cases they have their data feeds available behind a REST API, this makes your life much simpler. 

In other cases you need to hack your way into some C++ or Java code and create the wrappers to access their data. It is always useful to poke around their documentations and ask the sales guys who come by your desk from time-to-time. 

If you don't have access to price and fundemental data through paid services, worry not! you can use number of R/Python tools to harvest data from Web APIs and websites, more on those in later posts.  

One of these free web APIs is a great product called [Quandl](https://www.quandl.com/). This is a Toronto based company that collects and aggregates data into time series format that can be accessed by all the popular data analysis tools such as R and Python. 

## Quandl Setup

Getting data from Quandl is as easy as 1-2-3

1) You can download data without registering; however, that will give you access to only a short window of data. The best way is to signup to the service and to get an API KEY
2) Install Quandl package from Github 
3) Find the Data Series and use Quandl function to pull the data 

Example below shows how to pull the Natural Gas EIA storage number and price data from Quandl  


First step is to setup the R package and do a test. 


```r
install.packages("devtools")
install.packages("dplyr")
library(devtools)
install_github('quandl/R-package')
library(Quandl)
mydata = Quandl("CHRIS/CME_NG1",trim_start="1983-03-30", trim_end="2015-10-20")
```

Second is to pull the data in this case front contract price and NYMEX volume data. 


```r
library(Quandl)
require(dplyr)
require(ggplot2)

mydata = Quandl("CHRIS/CME_NG1",trim_start="1983-03-30", trim_end="2015-10-20")

head(mydata)
```

```
##         Date  Open  High   Low  Last Change Settle Volume Open Interest
## 1 2015-10-20 2.453 2.499 2.444 2.487  0.034  2.476 134766        116464
## 2 2015-10-19 2.464 2.481 2.435 2.457  0.012  2.442  93545        129977
## 3 2015-10-16 2.453 2.466 2.410 2.427  0.023  2.430 118802        147299
## 4 2015-10-15 2.538 2.578 2.449 2.451  0.065  2.453 176749        155301
## 5 2015-10-14 2.499 2.539 2.472 2.537  0.020  2.518 140714        177471
## 6 2015-10-13 2.544 2.555 2.487 2.498  0.037  2.498 146978        194342
```

```r
mydata<-mydata %>% dplyr::mutate(Date=as.Date(Date,format="%Y-%m-%d")) %>% dplyr::arrange(-desc(Date)) %>% dplyr::mutate(Year=lubridate::year(Date)) %>% dplyr::select(Date,Settle,Year,Volume) %>% dplyr::filter(Date > as.Date("2012-01-01","%Y-%m-%d"))
ggplot(data=mydata, aes(x=Settle, y=Volume, color=factor(Year)))+
geom_point()+ggtitle("")+stat_smooth()
```

```
## geom_smooth: method="auto" and size of largest group is <1000, so using loess. Use 'method = x' to change the smoothing method.
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 










