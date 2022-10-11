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
library(tidyverse)
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
refers to the date on witch the news is published greater than the
specified value. The data type is string and the format should be
`yyyy-mm-dd`. The third parameter y`limit` limits the number of news
will be returned from the function. The data type is integer. This
function returns a structured list including multiple information
regarding each news such as publisher, author, title, keywords, etc.

``` r
GetNews = function(ticker, published, limit) {
  ticker = toupper(ticker)
  # generate the URL
  base = "https://api.polygon.io/v2/reference/"
  Endpoint = "news"
  call = paste(base,Endpoint,"?ticker=",ticker,"&published_utc.gt=",published,
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
head(Price)
```

    ##   Company TradingVolumn VolumnWeightedAverage OpenPrice ClosePrice HighestPrice
    ## 1   Apple      77287356              146.9910   145.935     146.80     148.1950
    ## 2   Apple      71447416              148.0368   147.550     148.56     148.7177
    ## 3   Apple      72434089              149.1182   148.270     148.99     149.8300
    ## 4   Apple     104803028              146.8324   149.120     146.77     149.2100
    ## 5   Apple     118931191              144.8875   144.810     144.98     146.9700
    ## 6   Apple      56699475              145.8064   144.685     145.64     146.5500
    ##   LowestPrice    Timestamp Transactions
    ## 1      145.81 1.626926e+12       480209
    ## 2      146.92 1.627013e+12       457247
    ## 3      147.70 1.627272e+12       489114
    ## 4      145.55 1.627358e+12       785621
    ## 5      142.54 1.627445e+12       829463
    ## 6      144.58 1.627531e+12       416123

Plots You should pull data from at least two calls to your obtaining
data function (possibly combining them into one) We have called the
function twice to obtain the data for Apple and Amazon companies. We
have then plotted the opening price for the stocks against the timestamp
for doing various analysis like identifying the trends etc.

``` r
ggplot(data = Price, aes(x = Timestamp, y = OpenPrice)) +
geom_line(aes(color = Company))
```

![](C:\Users\hechu\OneDrive\Courses\ST%20558\Project\Project%202\ST-558-Project-2\README_files/figure-gfm/unnamed-chunk-29-1.png)<!-- -->
You should create at least one new variable that is a function of other
variables : Here, we have created a new variable “PriceVariation” which
is the difference between the highest and lowest price of the stocks of
a company per day. This variation is then plotted against the timestamp
and following graph is obtaine.

``` r
priceUpdated <- Price %>%
  filter(Company == "Apple") %>%
  mutate(PriceVariation = (HighestPrice-LowestPrice)) %>%
  arrange(desc(PriceVariation))

ggplot(data = priceUpdated, aes(x = Timestamp, y = PriceVariation)) +
  geom_col()
```

![](C:\Users\hechu\OneDrive\Courses\ST%20558\Project\Project%202\ST-558-Project-2\README_files/figure-gfm/unnamed-chunk-30-1.png)<!-- -->

``` r
g <- ggplot(data = Price_Apple, aes(x = OpenPrice))
g + geom_histogram()
```

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](C:\Users\hechu\OneDrive\Courses\ST%20558\Project\Project%202\ST-558-Project-2\README_files/figure-gfm/unnamed-chunk-31-1.png)<!-- -->

Next, let’s create some contingency tables to summarize the key words
for different types of companies in the news. We are interested in the
key words in news for two types of companies: technology company and
finance company. We selected `AAPL` and `AMZN` as technical company
samples, `JPM` and `WFC` as financial company samples.

``` r
News_AAPL = GetNews("aapl","2021-07-01",500)
wordvec_AAPL = tolower(unlist(News_AAPL$keywords))

News_AMZN = GetNews("amzn","2021-07-01",500)
wordvec_AMZN = tolower(unlist(News_AMZN$keywords))

News_JPM = GetNews("jpm","2021-07-01",500)
wordvec_JPM = tolower(unlist(News_JPM$keywords))

News_WFC = GetNews("wfc","2021-07-01",500)
worvec_WFC = tolower(unlist(News_WFC$keywords))
```

We grabbed the keywords in the first 500 news of the corresponding
companies from 2021-07-01, save it as a string vector. Then we want to
search for the keywords of `tech`, `investing`, and `markets` in the
news from the four companies. We first create a `keywords` vector to
save the word list. Next we search for the keywords in the corresponding
string vectors, and create four contingency tables accordingly.

``` r
keywords = c("tech", "investing","markets","eurozone")
wordcount_AAPL = wordvec_AAPL[wordvec_AAPL %in% keywords]
wordcount_AMZN = wordvec_AMZN[wordvec_AMZN %in% keywords]
wordcount_JPM = wordvec_JPM[wordvec_JPM %in% keywords]
wordcount_WFC = worvec_WFC[worvec_WFC %in% keywords]

table(wordcount_AAPL)
```

    ## wordcount_AAPL
    ## investing   markets      tech 
    ##       132        67        67

``` r
table(wordcount_AMZN)
```

    ## < table of extent 0 >

``` r
table(wordcount_JPM)
```

    ## < table of extent 0 >

``` r
table(wordcount_WFC)
```

    ## < table of extent 0 >

It can be seen that there is a clear association between the keywords
and company types. Two technical companies all includes keyword of
`tech`, while either of the financial companies has fewer frequency of
`tech`. On the other hand, financial terminology such as `eurozone` has
higher frequencies in financial companies.
