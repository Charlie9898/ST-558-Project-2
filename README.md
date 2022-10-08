Project2
================
Chuanni He
2022-10-03

# Backgroun Information

This project is prepared by Aniket Anil Walimbe and Chuanni He. It
created a vignette for reading and summarizing data from one of the
[Financial data API](https://polygon.io/docs/getting-started).

# Requirements

To use the functions for interacting with the Financial API, We used the
following packages:  
\* `httr`: Tools for Working with URLs and HTTP.  
\* `jsonlite`: A JSON parser/generator optimized for the web. Its main
strength is that it implements a bidirectional mapping between JSON data
and the most important R data types.  
\* `ggplot2`: A system for declaratively creating graphics, based on The
Grammar of Graphics.

``` r
# Required library and key information
key = "EByG3OEye377G2rI0LRah2CUjGplA7TB"
library(httr)
library(jsonlite)
library(ggplot2)
```

# API Interaction Functions

This section talks about how we define the functions to interact with
the Financial API, as well as some helper functions working with data
manipulation.

`Aggregates`

Our first function is to obtain the stock price for a stock over a given
date range in custom time window sizes. This function interacts with the
`Aggregates (Bars)` endpoint of the API. `stock` parameter refers to the
ticker of the referred company. For example, the stock ticker for Apple
Inc. is `AAPL`. Either lower-case or capital letters are acceptable. The
second parameter `multiplier` refers to the time multiplier for the
acquired data. For example, if the `timespan` is `day`, then
`multiplier` of `5` indicates that 5-minute bars will be obtained.
`multiplier` should be integer values. The last parameter `timespan`
refers to the size of the time window. The possible values are `minute`,
`hour`, `day`, `week`, `month`, `quarter`, `year`. Either lower-case or
capital letters are acceptable. This function will return the close
price in the given time period between 2021-7-22 to 2022-7-22 with a
maximum data length of 200 records. The returned data type is data
frame.

``` r
Aggregates = function(ticker,multiplier, timespan) {
  # generate the URL
  base = "https://api.polygon.io/v2/aggs/"
  Endpoint = "ticker"
  ticker = toupper(ticker)
  timespan = tolower(timespan)
  call = paste(base,Endpoint,"/",ticker,"/range/",multiplier,"/",timespan,
               "/2021-07-22/2022-07-22?adjusted=true&sort=asc&limit=200&apiKey=",
               key, sep = "")
  # Acquire the data
  get = GET(call)
  parsed = fromJSON(rawToChar(get$content))
  result = parsed$results
  colnames(result) = c("TradingVolumn", "VolumnWeightedAverage", 
                       "OpenPrice","ClosePrice", "HighestPrice",
                       "LowestPrice","Timestamp","Transactions")
  
  return(result)
}
```

`GetNews`

The second function acquires the most recent news articles relating to a
stock ticker. The summary of the article and the link to the original
article are also included. Three parameters are included in this
function. The `ticker` is the stock ticker of the company. Either
lower-case or capital letters are acceptable. The `published` parameter
refers to the date on witch the news is published. The data type is
string and the format should be `yyyy-mm-dd`. The third parameter
y`limit` limits the number of news will be returned from the function.
The data type is integer. This function returns a structured list
including multiple information regarding each news such as publisher,
author, title, keywords, etc.

``` r
GetNews = function(ticker, published, limit) {
  ticker = toupper(ticker)
  # generate the URL
  base = "https://api.polygon.io/v2/reference/"
  Endpoint = "news"
  call = paste(base,Endpoint,"?ticker=",ticker,"&published_utc=",published,
               "&limit=",limit, "&apiKey=", key, sep = "")
  # Acquire the data
  get = GET(call)
  parsed = fromJSON(rawToChar(get$content))
  # We removed the publisher information
  result = parsed$results[,-2]
  return(result)
}
```

# Data Exploration

Now that we can interact with a few of the endpoints of the Financial
API, let’s get some data from them.  
First, let’s pull the stock price of Apple Inc. (the ticker is `AAPL`)
and Amazon.com Inc. (the ticker is `AMZN`) during 2021-7-22 and
2022-7-22. We set the timeframe as 1 day (`multiplier=1`,
`timespan="day"`), then, we call the function
`Aggregates("aapl", 1, "day")` and `Aggregates("amzn", 1, "day")`. Then
combine them together by adding a `Company` variable.

``` r
Price_Apple = Aggregates("aapl", 1, "day")
Price_Amazon = Aggregates("amzn", 1, "day")

Price_Apple = data.frame(Company="Apple",Price_Apple)
Price_Amazon = data.frame(Company="Amazon",Price_Amazon)
Price = rbind(Price_Apple,Price_Amazon)
```

``` r
News = GetNews("aapl","2021-07-01",10)
```
