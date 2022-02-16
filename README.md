
<!-- README.md is generated from README.Rmd. Please edit that file -->

# lmForc Helper Functions

``` r
library(lmForc)
```

Assorted functions for working with lmForc Forecast objects. The end
game is to integrate some of these functions into the lmForc package and
turn the rest of these functions into a standalone R package.

## Subsetting Example Dataset

Examples of lmForc subsetting functions utilize the following stylized
dataset. This example dataset contains two forecasts with one quarter
ahead forecasts of the same variable. Note that both forecasts have
identical `future`, `realized`, and `h_ahead` values, but that `origin`
dates of the last two forecasts differ. These example forecasts can be
thought of as two different model forecasts of the same variable.

``` r
forc1_1h <- Forecast(
  origin = as.Date(c("2010-02-17", "2010-05-14", "2010-07-22", "2010-12-05", "2011-03-10")),
  future = as.Date(c("2010-06-30", "2010-09-30", "2010-12-31", "2011-03-31", "2011-06-30")),
  forecast = c(4.27, 3.36, 4.78, 5.45, 5.12),
  realized = c(4.96, 4.17, 4.26, 4.99, 5.38),
  h_ahead = 1
)

forc2_1h <- Forecast(
  origin = as.Date(c("2010-02-17", "2010-05-14", "2010-07-22", "2010-12-22", "2011-03-27")),
  future = as.Date(c("2010-06-30", "2010-09-30", "2010-12-31", "2011-03-31", "2011-06-30")),
  forecast = c(4.01, 3.89, 3.31, 4.33, 4.61),
  realized = c(4.96, 4.17, 4.26, 4.99, 5.38),
  h_ahead = 1
)
```

## Subsetting Functions Overview

``` r
source("lmForc_subset.R")
```

The Forecast class has a built in method for subsetting a single
Forecast object. This subsetting method takes numeric or logical values
and follows subsetting conventions that are present throughout the R
language.

``` r
forc1_1h[2:4]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-05-14 2010-09-30     3.36     4.17
#> 2 2010-07-22 2010-12-31     4.78     4.26
#> 3 2010-12-05 2011-03-31     5.45     4.99

forc1_1h[origin(forc1_1h) >= as.Date("2010-12-31")]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2011-03-10 2011-06-30     5.12     5.38
```

However, one often ends up working with multiple Forecast objects
whether that be different model forecasts for the same forecast horizon,
one model forecast for varying forecast horizons, or both. The lmForc
convention for working with multiple Forecast objects is to put them
into a list. The following subsetting functios provide a way to subset
multiple Forecast objects by various conditions.

## subset_forcs

The simplest way to subset multiple Forecast objects is via the
`subset_forcs()` function. The function takes a list of Forecast objects
and a numeric or logical index. All forecasts in the list are subset by
the whatever is passed to the `index` argument.

``` r
forcs <- list(forc1_1h, forc2_1h)

subset_forcs(forcs, 1:4)
#> [[1]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-02-17 2010-06-30     4.27     4.96
#> 2 2010-05-14 2010-09-30     3.36     4.17
#> 3 2010-07-22 2010-12-31     4.78     4.26
#> 4 2010-12-05 2011-03-31     5.45     4.99
#> 
#> [[2]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-02-17 2010-06-30     4.01     4.96
#> 2 2010-05-14 2010-09-30     3.89     4.17
#> 3 2010-07-22 2010-12-31     3.31     4.26
#> 4 2010-12-22 2011-03-31     4.33     4.99
 
subset_forcs(forcs, origin(forc1_1h) >= as.Date("2010-12-31"))
#> [[1]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2011-03-10 2011-06-30     5.12     5.38
#> 
#> [[2]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2011-03-27 2011-06-30     4.61     5.38
```

## subset_bytime

One may wish to compare forecasts over a specific time horizon. The
`subset_bytime()` allows the user to subset multiple forecasts based on
origin or future values. After choosing whether to subset by origin or
future values and passing a single time object or a vector of time
objects to the `values` argument, all forecasts in a list of Forecast
objects are subset by `values`.

``` r
forcs <- list(forc1_1h, forc2_1h)
 
subset_bytime(forcs, values = as.Date("2010-05-14"), slot = "origin")
#> [[1]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-05-14 2010-09-30     3.36     4.17
#> 
#> [[2]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-05-14 2010-09-30     3.89     4.17

subset_bytime(
  forcs, 
  values = as.Date(c("2010-09-30", "2010-12-31", "2011-03-31")), 
  slot = "future"
)
#> [[1]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-05-14 2010-09-30     3.36     4.17
#> 2 2010-07-22 2010-12-31     4.78     4.26
#> 3 2010-12-05 2011-03-31     5.45     4.99
#> 
#> [[2]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-05-14 2010-09-30     3.89     4.17
#> 2 2010-07-22 2010-12-31     3.31     4.26
#> 3 2010-12-22 2011-03-31     4.33     4.99
```

## subset_identical

When comparing multiple forecasts, it is imperative that forecast
accuracy is compared across an identical time period. This can become
complicated if the origin and future values of multiple forecasts do not
always align. The `subset_identical()` function finds all overlapping
origin or future values in a list of Forecast objects and subsets each
of the forecasts to these overlapping values. This leaves the user with
a list of Forecasts that have either identical origin values or
identical future values depending on what the user passes to the `slot`
argument.

``` r
forcs <- list(forc1_1h, forc2_1h)

subset_identical(forcs, slot = "origin")
#> [[1]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-02-17 2010-06-30     4.27     4.96
#> 2 2010-05-14 2010-09-30     3.36     4.17
#> 3 2010-07-22 2010-12-31     4.78     4.26
#> 
#> [[2]]
#> h_ahead = 1 
#> 
#>       origin     future forecast realized
#> 1 2010-02-17 2010-06-30     4.01     4.96
#> 2 2010-05-14 2010-09-30     3.89     4.17
#> 3 2010-07-22 2010-12-31     3.31     4.26
```

## Transformation Functions Overview

``` r
source("lmForc_transform.R")
```

When compiling multiple Forecast objects, there are two general formats
that a list of forecasts can take.

# Helper Functions Overview
