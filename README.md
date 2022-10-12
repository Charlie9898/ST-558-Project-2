Project2
================
Chuanni He
2022-10-12

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
\* `tidyverse`: An opinionated collection of R packages designed for
data science. All packages share an underlying design philosophy,
grammar, and data structures.

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

## `Aggregates`

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

## `GetNews`

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
API, let’s get some data from them. Here, we defined the scope of this
project to investigate two types of companies: technical companies and
financial companies. The specific companies we will focus on are: \*
Apple Inc. (`AAPL`) \* Amazon Inc. (`AMZN`) \* JPMorgan Chase & Co
(`JPM`) \* Wells Fargo & Co (`WFC`)

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

# Exploratory Data Analysis

## Trend for Openning Price

We have called the function twice to obtain the data for Apple and
Amazon companies. We have then plotted the opening price for the stocks
against the timestamp for doing various analysis like identifying the
trends etc.

``` r
ggplot(data = Price, aes(x = Timestamp, y = OpenPrice)) +
geom_line(aes(color = Company))
```

![Open price for Amazon and
Apple](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-5-1.png)

From the plot above, we can found that there is a decreasing major trend
for Amazon and a rising trend for Apple. In addition, the two companies
share similar fluctuation trend in a small timeframe (can be interpreted
as being impacted by the entire market). When there is a rising thrend,
both of them tend to have a increasing pattern, and vise versa.

## Creating New Variables

Next, we have created a new variable `PriceVariation` which is the
difference between the highest and lowest price of the stocks of a
company per day. The new variable represents the fluctuation range of
the corresponding company. This variation is then plotted against the
timestamp and following graph is obtained.

``` r
priceUpdated <- Price %>%
  filter(Company == "Apple") %>%
  mutate(PriceVariation = (HighestPrice-LowestPrice)) %>%
  arrange(desc(PriceVariation))

ggplot(data = priceUpdated, aes(x = Timestamp, y = PriceVariation, color = "blue")) +
  geom_col() + 
  labs(title = "Plot for Price Variation vs Time")
```

![Plot for Price Variation vs
Time](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-6-1.png)

From the plot above, we can find that there is no clear pattern for the
daily fluctuation for Apple Inc. However, in the past year, there is a
minor increasing trend for the daily fluctuation, indicating that the
stock for Apple is becoming more insteable.

## Numerical Summaries for Categorical Variables

Since we limited our scope of this project to focus on four companies,
here for each company, we calculated the summary statistics for the
`PriceVariation` of all the four companies.

``` r
Price_JPMORGAN = Aggregates("jpm", 1, "day")
Price_WELLSFARGO = Aggregates("wfc", 1, "day")
Price_JPMORGAN = data.frame(Company="JPMorgan",Price_JPMORGAN)
Price_WELLSFARGO = data.frame(Company="Wells Fargo",Price_WELLSFARGO)
Price = rbind(Price,Price_JPMORGAN,Price_WELLSFARGO)

priceNew <- Price %>%
  mutate(PriceVariation = (HighestPrice-LowestPrice))

Price_Summary = priceNew %>% group_by(Company) %>%
                summarize("Min." = min(PriceVariation),
                          "1st Quartile" = quantile(PriceVariation, 0.25, na.rm=TRUE),
                          "Median" = quantile(PriceVariation, 0.5, na.rm=TRUE),
                          "Mean" = mean(PriceVariation, na.rm=TRUE),
                          "3rd Quartile" = quantile(PriceVariation, 0.75, na.rm=TRUE),
                          "Max" = max(PriceVariation),
                          "Std. Dev." = sd(PriceVariation, na.rm=TRUE))
Price_Summary
```

    ## # A tibble: 4 × 8
    ##   Company      Min. `1st Quartile` Median  Mean `3rd Quartile`   Max `Std. Dev.`
    ##   <chr>       <dbl>          <dbl>  <dbl> <dbl>          <dbl> <dbl>       <dbl>
    ## 1 Amazon      1.09           2.72    3.82  4.21           5.38 12.2        1.93 
    ## 2 Apple       1.17           2.38    3.34  3.61           4.49 10.8        1.72 
    ## 3 JPMorgan    1.33           2.21    2.85  3.08           3.72  6.90       1.08 
    ## 4 Wells Fargo 0.485          0.999   1.19  1.32           1.57  3.78       0.533

Wells Fargo has the least daily variation, indicating that the stock is
very steable in a daily timeframe. Technical companies have overall
higher daily fluctuation than financial companies.

We also considered the Open Price and Close price variables and found
out the mean, median, mode and IQR for both the variables. Using the
group_by function, this data was obtained company wise. The table shows
these values below :

``` r
Price_StatsOP <- Price %>%
  group_by(Company) %>%
  summarise(Avg_Open_Price = mean(OpenPrice), Std_dev = sd(OpenPrice), Median = median(OpenPrice), IQR = IQR(OpenPrice))

Price_StatsCP <- Price %>%
  group_by(Company) %>%
  summarise(Avg_Close_Price = mean(ClosePrice), Std_dev = sd(ClosePrice), Median = median(ClosePrice), IQR = IQR(ClosePrice))

  
Price_StatsOP
```

    ## # A tibble: 4 × 5
    ##   Company     Avg_Open_Price Std_dev Median   IQR
    ##   <chr>                <dbl>   <dbl>  <dbl> <dbl>
    ## 1 Amazon               163.    12.2   166.  14.0 
    ## 2 Apple                159.    11.7   159.  21.5 
    ## 3 JPMorgan             153.    13.5   157.  19.6 
    ## 4 Wells Fargo           49.7    3.58   49.2  4.13

``` r
Price_StatsCP
```

    ## # A tibble: 4 × 5
    ##   Company     Avg_Close_Price Std_dev Median   IQR
    ##   <chr>                 <dbl>   <dbl>  <dbl> <dbl>
    ## 1 Amazon                163.    12.5   165.  15.0 
    ## 2 Apple                 160.    11.7   160.  21.5 
    ## 3 JPMorgan              153.    13.4   157.  18.0 
    ## 4 Wells Fargo            49.7    3.57   48.9  4.14

Some findings can be concluded form the data above. First, Amazon has
slightly higher open and close price than Apple. Both of them have
almost the same standard deviation regarding the open and close price.
In addition, Apple has significantly higher IQR than Amazon.

## Contingency Tables

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
    ##       134        62        69

``` r
table(wordcount_AMZN)
```

    ## wordcount_AMZN
    ##  eurozone investing   markets      tech 
    ##         1       190        60        26

``` r
table(wordcount_JPM)
```

    ## wordcount_JPM
    ##  eurozone investing   markets      tech 
    ##         4       118        27         4

``` r
table(wordcount_WFC)
```

    ## wordcount_WFC
    ##  eurozone investing   markets      tech 
    ##         3       129        29         2

It can be seen that there is a clear association between the keywords
and company types. Two technical companies all includes keyword of
`tech`, while either of the financial companies has fewer frequency of
`tech`. On the other hand, financial terminology such as `eurozone` has
higher frequencies in financial companies.

## Investigating the Keyword Count for Different Companies

In this plot, we have first converted the `wordcount_AAPL` list to a
tibble and then, plotted the keywords present in the table according to
their values.

``` r
df_temp <- as_tibble(wordcount_AAPL)
ggplot(data = df_temp, aes(x = value)) +
  geom_bar(fill="blue") + 
  labs(x = "Apple", title = "Bar plot for Apple data")
```

![Bar plot for Apple
data](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-11-1.png)

It can be found that `investing` is a high frequency word over Apple,
indicating that the news are potentially focusing on financial
investment analysis or suggestions, etc.

We also made a box plot to reveal the closing price for Amazon and
Apple.

``` r
ggplot(data = Price, aes(x = Company, y = ClosePrice)) +
  geom_boxplot(fill = "grey")+
  labs(y = "Close Price", title = "Box Plot for Closing Price" )
```

![Box Plot for Closing
Price](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-12-1.png)

As identified in the data summary, Amazon has slightly higher average
close price than Apple, while Apple has higher IQR than Amazon.

The scatter plot was also developed to investigate the relationship
between open price and closing price. We plotted the `OpenPrice` and
`ClosePrice` for a company against each other. Using the facet_wrap()
function, we can see the subplots for 2 companies.

``` r
ggplot(data = Price, aes(x = OpenPrice, y = ClosePrice)) +
  geom_point(color = "blue") +
  labs(x = "Open Price" , y = "Close Price", title = "Scatter Plot : Open Price Vs Close Price") +
  facet_wrap(~ Company)
```

![Scatter Plot : Open Price Vs Close
Price](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-13-1.png)

There is a very strong linear relationship between open price and close
price for both companies. We can assume that the overall trend of the
stock market have a dominant effect over the company, rather than the
daily fluctuation.

Finally, the histogram of the Transactions for Amazon stocks is
developed as well.

``` r
ggplot(data = Price_Amazon, aes(x= Transactions)) +
  geom_histogram(fill="lightblue", color="black", size = 5, binwidth = 10000)+
  labs(title = "Histogram for Transactions - Amazon")
```

![Histogram for Transactions -
Amazon](https://raw.githubusercontent.com/Charlie9898/ST-558-Project-2/main/README_files/figure-gfm/unnamed-chunk-14-1.png)

The daily transaction has a bell curve distribution with a long tail.
Most of the daily transaction falls in the range between (0, 250,000).

# Wrap-Up

In this vignette, we developed several functions to extract stock market
data from the Financial Data API. We accessed the stock price and news
data for Apple, Amazon, JPMorgan, and Wells Fargo company, then
performed some basic EDA to explore some patterns. There are some
initial findings. For example, we found that financial companies have a
more stable stock price than technical companies. While Technical
companies have more exposure in press than financial companies.
