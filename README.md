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
\* `httr`: Tools for Working with URLs and HTTP \* `jsonlite`: A JSON
parser/generator optimized for the web. Its main strength is that it
implements a bidirectional mapping between JSON data and the most
important R data types.

``` r
# Required library and key information
key = "EByG3OEye377G2rI0LRah2CUjGplA7TB"
library(httr)
library(jsonlite)
```

# API Interaction Functions

This section talks about how we define the functions to interact with
the Financial API, as well as some helper functions working with data
manipulation.

`Aggregates`

Our first function is to obtain the aggregate bars for a stock over a
given date range in custom time window sizes. This function interacts
with the `Aggregates (Bars)` endpoint of the API. `stock` parameter
refers to the ticker of the referred company. For example, the stock
ticker for Apple Inc. is `AAPL`. Either lower-case or capital letters
are acceptable. The second parameter `multiplier` refers to the time
multiplier for the acquired data. For example, if the `timespan` is
`day`, then `multiplier` of `5` indicates that 5-minute bars will be
obtained. `multiplier` should be integer values. The last parameter
`timespan` refers to the size of the time window. The possible values
are `minute`, `hour`, `day`, `week`, `month`, `quarter`, `year`. Either
lower-case or capital letters are acceptable.

``` r
# timespan = (minute,hour,day,week,month,quarter,year)
Aggregates = function(stock,multiplier, timespan) {
  base = "https://api.polygon.io/v2/aggs/"
  Endpoint = "ticker"
  stock = toupper(stock)
  timespan = tolower(timespan)
  call = paste(base,Endpoint,"/",stock,"/range/",multiplier,"/",timespan,
               "/2021-07-22/2022-07-22?adjusted=true&sort=asc&limit=200&apiKey=",
               key, sep = "")
  
  get = GET(call)
  content = content(get)
  return(content)
}

stock = "aapl"
multiplier = 1
timespan = "day"
content = Aggregates(stock,multiplier,timespan)
```

``` r
for (i in 1:length(content$results)) {
  print(content$results[[i]]$c)
}
```

    ## [1] 146.8
    ## [1] 148.56
    ## [1] 148.99
    ## [1] 146.77
    ## [1] 144.98
    ## [1] 145.64
    ## [1] 145.86
    ## [1] 145.52
    ## [1] 147.36
    ## [1] 146.95
    ## [1] 147.06
    ## [1] 146.14
    ## [1] 146.09
    ## [1] 145.6
    ## [1] 145.86
    ## [1] 148.89
    ## [1] 149.1
    ## [1] 151.12
    ## [1] 150.19
    ## [1] 146.36
    ## [1] 146.7
    ## [1] 148.19
    ## [1] 149.71
    ## [1] 149.62
    ## [1] 148.36
    ## [1] 147.54
    ## [1] 148.6
    ## [1] 153.12
    ## [1] 151.83
    ## [1] 152.51
    ## [1] 153.65
    ## [1] 154.3
    ## [1] 156.69
    ## [1] 155.11
    ## [1] 154.07
    ## [1] 148.97
    ## [1] 149.55
    ## [1] 148.12
    ## [1] 149.03
    ## [1] 148.79
    ## [1] 146.06
    ## [1] 142.94
    ## [1] 143.43
    ## [1] 145.85
    ## [1] 146.83
    ## [1] 146.92
    ## [1] 145.37
    ## [1] 141.91
    ## [1] 142.83
    ## [1] 141.5
    ## [1] 142.65
    ## [1] 139.14
    ## [1] 141.11
    ## [1] 142
    ## [1] 143.29
    ## [1] 142.9
    ## [1] 142.81
    ## [1] 141.51
    ## [1] 140.91
    ## [1] 143.76
    ## [1] 144.84
    ## [1] 146.55
    ## [1] 148.76
    ## [1] 149.26
    ## [1] 149.48
    ## [1] 148.69
    ## [1] 148.64
    ## [1] 149.32
    ## [1] 148.85
    ## [1] 152.57
    ## [1] 149.8
    ## [1] 148.96
    ## [1] 150.02
    ## [1] 151.49
    ## [1] 150.96
    ## [1] 151.28
    ## [1] 150.44
    ## [1] 150.81
    ## [1] 147.92
    ## [1] 147.87
    ## [1] 149.99
    ## [1] 150
    ## [1] 151
    ## [1] 153.49
    ## [1] 157.87
    ## [1] 160.55
    ## [1] 161.02
    ## [1] 161.41
    ## [1] 161.94
    ## [1] 156.81
    ## [1] 160.24
    ## [1] 165.3
    ## [1] 164.77
    ## [1] 163.76
    ## [1] 161.84
    ## [1] 165.32
    ## [1] 171.18
    ## [1] 175.08
    ## [1] 174.56
    ## [1] 179.45
    ## [1] 175.74
    ## [1] 174.33
    ## [1] 179.3
    ## [1] 172.26
    ## [1] 171.14
    ## [1] 169.75
    ## [1] 172.99
    ## [1] 175.64
    ## [1] 176.28
    ## [1] 180.33
    ## [1] 179.29
    ## [1] 179.38
    ## [1] 178.2
    ## [1] 177.57
    ## [1] 182.01
    ## [1] 179.7
    ## [1] 174.92
    ## [1] 172
    ## [1] 172.17
    ## [1] 172.19
    ## [1] 175.08
    ## [1] 175.53
    ## [1] 172.19
    ## [1] 173.07
    ## [1] 169.8
    ## [1] 166.23
    ## [1] 164.51
    ## [1] 162.41
    ## [1] 161.62
    ## [1] 159.78
    ## [1] 159.69
    ## [1] 159.22
    ## [1] 170.33
    ## [1] 174.78
    ## [1] 174.61
    ## [1] 175.84
    ## [1] 172.9
    ## [1] 172.39
    ## [1] 171.66
    ## [1] 174.83
    ## [1] 176.28
    ## [1] 172.12
    ## [1] 168.64
    ## [1] 168.88
    ## [1] 172.79
    ## [1] 172.55
    ## [1] 168.88
    ## [1] 167.3
    ## [1] 164.32
    ## [1] 160.07
    ## [1] 162.74
    ## [1] 164.85
    ## [1] 165.12
    ## [1] 163.2
    ## [1] 166.56
    ## [1] 166.23
    ## [1] 163.17
    ## [1] 159.3
    ## [1] 157.44
    ## [1] 162.95
    ## [1] 158.52
    ## [1] 154.73
    ## [1] 150.62
    ## [1] 155.09
    ## [1] 159.59
    ## [1] 160.62
    ## [1] 163.98
    ## [1] 165.38
    ## [1] 168.82
    ## [1] 170.21
    ## [1] 174.07
    ## [1] 174.72
    ## [1] 175.6
    ## [1] 178.96
    ## [1] 177.77
    ## [1] 174.61
    ## [1] 174.31
    ## [1] 178.44
    ## [1] 175.06
    ## [1] 171.83
    ## [1] 172.14
    ## [1] 170.09
    ## [1] 165.75
    ## [1] 167.66
    ## [1] 170.4
    ## [1] 165.29
    ## [1] 165.07
    ## [1] 167.4
    ## [1] 167.23
    ## [1] 166.42
    ## [1] 161.79
    ## [1] 162.88
    ## [1] 156.8
    ## [1] 156.57
    ## [1] 163.64
    ## [1] 157.65
    ## [1] 157.96
    ## [1] 159.48
    ## [1] 166.02
    ## [1] 156.77
