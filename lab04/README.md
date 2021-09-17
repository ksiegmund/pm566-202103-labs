lab 4
================
ks
9/17/2021

## Lab week 4

``` r
if (!require(data.table)) {install.packages("data.table")}
```

    ## Loading required package: data.table

``` r
library(data.table)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ ggplot2 3.3.5     ✓ purrr   0.3.4
    ## ✓ tibble  3.1.4     ✓ dplyr   1.0.7
    ## ✓ tidyr   1.1.3     ✓ stringr 1.4.0
    ## ✓ readr   2.0.1     ✓ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## x dplyr::between()   masks data.table::between()
    ## x dplyr::filter()    masks stats::filter()
    ## x dplyr::first()     masks data.table::first()
    ## x dplyr::lag()       masks stats::lag()
    ## x dplyr::last()      masks data.table::last()
    ## x purrr::transpose() masks data.table::transpose()

``` r
library(leaflet)
```

### 1. Read in the data

First read the data into a data.table.

``` r
if (!file.exists("~kims/GitHub/pm566-202103-labs/lab03/met_all.gz")) {
download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz", "met_all.gz", method="libcurl", timeout = 60)
}
met <- data.table::fread("~kims/GitHub/pm566-202103-labs/lab03/met_all.gz")
```

## 2. Prepare the data

``` r
#Remove temperatures less than -17C.
met <- met[temp>-17]

#Make sure there are no missing data in the key variables coded as 9999, 999, etc
#temp, rh, wind.sp, vis.dist, dew.point, lat, lon, and elev.
met[,range(temp,na.rm=T)]
```

    ## [1] -3 56

``` r
met[,range(rh,na.rm=T)]
```

    ## [1]   0.8334298 100.0000000

``` r
met[,range(wind.sp,na.rm=T)]
```

    ## [1]  0 36

``` r
met[,range(vis.dist,na.rm=T)]
```

    ## [1]      0 160000

``` r
met[,range(dew.point,na.rm=T)]
```

    ## [1] -37.2  36.0

``` r
met[,range(lat,na.rm=T)]
```

    ## [1] 24.550 48.941

``` r
met[,range(lon,na.rm=T)]
```

    ## [1] -124.290  -68.313

``` r
met[,range(elev,na.rm=T)]
```

    ## [1]  -13 9999

``` r
met[elev==9999.0, elev:= NA ]

#Generate a date variable using the functions as.Date() (hint: You will need the following to create a date paste(year, month, day, sep = "-")).
met[,ymd := as.Date(paste(year, month, day, sep = "-"))]

#Using the data.table::week function, keep the observations of the first week of the month.
met[,table(week(ymd))]
```

    ## 
    ##     31     32     33     34     35 
    ## 297259 521600 527922 523847 446576

``` r
met <- met[ week(ymd) == 31]
dim(met)
```

    ## [1] 297259     31

``` r
#Compute the mean by station of the variables temp, rh, wind.sp, vis.dist, dew.point, lat, lon, and elev.
met_avg <- met[,.(
   temp      = mean(temp,na.rm=T),
   rh        = mean(rh,na.rm=T),
   wind.sp   = mean(wind.sp,na.rm=T),
   vis.dist  = mean(vis.dist,na.rm=T),
   dew.point = mean(dew.point,na.rm=T),
   lat       = mean(lat,na.rm=T),
   lon       = mean(lon,na.rm=T),
   elev      = mean(elev,na.rm=T)
)]
#Create a region variable for NW, SW, NE, SE based on lon = -98.00 and lat = 39.71 degrees
#Create a categorical variable for elevation as in the lecture slides
```
