<!-- README.md is generated from README.Rmd. Please edit that file -->
[![Build Status](https://travis-ci.org/mpadge/bikedata.svg)](https://travis-ci.org/mpadge/bikedata) [![Project Status: Concept - Minimal or no implementation has been done yet.](http://www.repostatus.org/badges/0.1.0/concept.svg)](http://www.repostatus.org/#concept) [![CRAN\_Status\_Badge](http://www.r-pkg.org/badges/version/bikedata)](http://cran.r-project.org/web/packages/bikedata) ![downloads](http://cranlogs.r-pkg.org/badges/grand-total/bikedata)

bikedata
========

R package to load data from public bicycle hire systems. Currently a proof-of-concept that only loads data from the New York City [citibike scheme](https://www.citibikenyc.com/).

### Installation

``` r
devtools::install_github("mpadge/bikedata")
```

    #> Loading bikedata

### Usage

``` r
library(bikedata)

# current verison
packageVersion("bikedata")
```

[citibike data](https://www.citibikenyc.com/system-data) first have to be downloaded:

``` r
dl_bikedata ()
```

And stored in a `postgres` database with

``` r
store_bikedata (data_dir="/data/data/junk")
```

Note that `store_bikedata()` will also download the data if they don't already exist.

These data can then be accessed with the `RPostgreSQL` package:

``` r
library(RPostgreSQL)
#> Loading required package: DBI
drv <- dbDriver("PostgreSQL")
citibike_con = dbConnect(drv, dbname = "nyc-citibike-data")

query <- function(sql, con = citibike_con) 
  fetch(dbSendQuery(con, sql), n = 1e8)
get_ntrips <- function ()
{
    n <- query("SELECT * FROM station_to_station_counts")
    stns <- sort (unique (n$start_station_id))
    # transpose to index is [from, ti]
    n <- t (array (n$count, dim=rep (length (stns), 2)))
    rownames (n) <- colnames (n) <- stns
    return (n)
}
nt <- get_ntrips ()
junk <- dbDisconnect (citibike_con)
cat (format (sum (nt), big.mark=",", scientific=FALSE), 
     "trips between", dim (nt)[1], "stations\n")
#> 5,876,302 trips between 421 stations
```

At present the `postgres` database has to be manually removed with

``` r
system ("dropdb nyc-citibike-data")
```

### Test Results

``` r
library(bikedata)
library(testthat)

date()

test_dir("tests/")
```
