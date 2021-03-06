03-lab-EDA
================
ks
9/10/2021

## Lab week 3

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
if (!file.exists("met_all.gz")) {
download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz", "data/met_all.gz", method="libcurl", timeout = 60)
}
met <- data.table::fread("met_all.gz")
```

### 2. Check the dimensions, header and footer.

``` r
dim(met)
```

    ## [1] 2377343      30

``` r
head(met)
```

    ##    USAFID  WBAN year month day hour min  lat      lon elev wind.dir wind.dir.qc
    ## 1: 690150 93121 2019     8   1    0  56 34.3 -116.166  696      220           5
    ## 2: 690150 93121 2019     8   1    1  56 34.3 -116.166  696      230           5
    ## 3: 690150 93121 2019     8   1    2  56 34.3 -116.166  696      230           5
    ## 4: 690150 93121 2019     8   1    3  56 34.3 -116.166  696      210           5
    ## 5: 690150 93121 2019     8   1    4  56 34.3 -116.166  696      120           5
    ## 6: 690150 93121 2019     8   1    5  56 34.3 -116.166  696       NA           9
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N     5.7          5      22000             5                 9
    ## 2:              N     8.2          5      22000             5                 9
    ## 3:              N     6.7          5      22000             5                 9
    ## 4:              N     5.1          5      22000             5                 9
    ## 5:              N     2.1          5      22000             5                 9
    ## 6:              C     0.0          5      22000             5                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp temp.qc dew.point
    ## 1:        N    16093           5       N          5 37.2       5      10.6
    ## 2:        N    16093           5       N          5 35.6       5      10.6
    ## 3:        N    16093           5       N          5 34.4       5       7.2
    ## 4:        N    16093           5       N          5 33.3       5       5.0
    ## 5:        N    16093           5       N          5 32.8       5       5.0
    ## 6:        N    16093           5       N          5 31.1       5       5.6
    ##    dew.point.qc atm.press atm.press.qc       rh
    ## 1:            5    1009.9            5 19.88127
    ## 2:            5    1010.3            5 21.76098
    ## 3:            5    1010.6            5 18.48212
    ## 4:            5    1011.6            5 16.88862
    ## 5:            5    1012.7            5 17.38410
    ## 6:            5    1012.7            5 20.01540

``` r
tail(met)
```

    ##    USAFID  WBAN year month day hour min    lat      lon elev wind.dir
    ## 1: 726813 94195 2019     8  31   18  56 43.650 -116.633  741       NA
    ## 2: 726813 94195 2019     8  31   19  56 43.650 -116.633  741       70
    ## 3: 726813 94195 2019     8  31   20  56 43.650 -116.633  741       NA
    ## 4: 726813 94195 2019     8  31   21  56 43.650 -116.633  741       10
    ## 5: 726813 94195 2019     8  31   22  56 43.642 -116.636  741       10
    ## 6: 726813 94195 2019     8  31   23  56 43.642 -116.636  741       40
    ##    wind.dir.qc wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc
    ## 1:           9              C     0.0          5      22000             5
    ## 2:           5              N     2.1          5      22000             5
    ## 3:           9              C     0.0          5      22000             5
    ## 4:           5              N     2.6          5      22000             5
    ## 5:           1              N     2.1          1      22000             1
    ## 6:           1              N     2.1          1      22000             1
    ##    ceiling.ht.method sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp
    ## 1:                 9        N    16093           5       N          5 30.0
    ## 2:                 9        N    16093           5       N          5 32.2
    ## 3:                 9        N    16093           5       N          5 33.3
    ## 4:                 9        N    14484           5       N          5 35.0
    ## 5:                 9        N    16093           1       9          9 34.4
    ## 6:                 9        N    16093           1       9          9 34.4
    ##    temp.qc dew.point dew.point.qc atm.press atm.press.qc       rh
    ## 1:       5      11.7            5    1013.6            5 32.32509
    ## 2:       5      12.2            5    1012.8            5 29.40686
    ## 3:       5      12.2            5    1011.6            5 27.60422
    ## 4:       5       9.4            5    1010.8            5 20.76325
    ## 5:       1       9.4            1    1010.1            1 21.48631
    ## 6:       1       9.4            1    1009.6            1 21.48631

How many rows and columns?

### 3. Look at variables

``` r
str(met)
```

    ## Classes 'data.table' and 'data.frame':   2377343 obs. of  30 variables:
    ##  $ USAFID           : int  690150 690150 690150 690150 690150 690150 690150 690150 690150 690150 ...
    ##  $ WBAN             : int  93121 93121 93121 93121 93121 93121 93121 93121 93121 93121 ...
    ##  $ year             : int  2019 2019 2019 2019 2019 2019 2019 2019 2019 2019 ...
    ##  $ month            : int  8 8 8 8 8 8 8 8 8 8 ...
    ##  $ day              : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ hour             : int  0 1 2 3 4 5 6 7 8 9 ...
    ##  $ min              : int  56 56 56 56 56 56 56 56 56 56 ...
    ##  $ lat              : num  34.3 34.3 34.3 34.3 34.3 34.3 34.3 34.3 34.3 34.3 ...
    ##  $ lon              : num  -116 -116 -116 -116 -116 ...
    ##  $ elev             : int  696 696 696 696 696 696 696 696 696 696 ...
    ##  $ wind.dir         : int  220 230 230 210 120 NA 320 10 320 350 ...
    ##  $ wind.dir.qc      : chr  "5" "5" "5" "5" ...
    ##  $ wind.type.code   : chr  "N" "N" "N" "N" ...
    ##  $ wind.sp          : num  5.7 8.2 6.7 5.1 2.1 0 1.5 2.1 2.6 1.5 ...
    ##  $ wind.sp.qc       : chr  "5" "5" "5" "5" ...
    ##  $ ceiling.ht       : int  22000 22000 22000 22000 22000 22000 22000 22000 22000 22000 ...
    ##  $ ceiling.ht.qc    : int  5 5 5 5 5 5 5 5 5 5 ...
    ##  $ ceiling.ht.method: chr  "9" "9" "9" "9" ...
    ##  $ sky.cond         : chr  "N" "N" "N" "N" ...
    ##  $ vis.dist         : int  16093 16093 16093 16093 16093 16093 16093 16093 16093 16093 ...
    ##  $ vis.dist.qc      : chr  "5" "5" "5" "5" ...
    ##  $ vis.var          : chr  "N" "N" "N" "N" ...
    ##  $ vis.var.qc       : chr  "5" "5" "5" "5" ...
    ##  $ temp             : num  37.2 35.6 34.4 33.3 32.8 31.1 29.4 28.9 27.2 26.7 ...
    ##  $ temp.qc          : chr  "5" "5" "5" "5" ...
    ##  $ dew.point        : num  10.6 10.6 7.2 5 5 5.6 6.1 6.7 7.8 7.8 ...
    ##  $ dew.point.qc     : chr  "5" "5" "5" "5" ...
    ##  $ atm.press        : num  1010 1010 1011 1012 1013 ...
    ##  $ atm.press.qc     : int  5 5 5 5 5 5 5 5 5 5 ...
    ##  $ rh               : num  19.9 21.8 18.5 16.9 17.4 ...
    ##  - attr(*, ".internal.selfref")=<externalptr>

### 4. Take a closer look

``` r
table(met$year)
```

    ## 
    ##    2019 
    ## 2377343

``` r
table(met$day)
```

    ## 
    ##     1     2     3     4     5     6     7     8     9    10    11    12    13 
    ## 75975 75923 76915 76594 76332 76734 77677 77766 75366 75450 76187 75052 76906 
    ##    14    15    16    17    18    19    20    21    22    23    24    25    26 
    ## 77852 76217 78015 78219 79191 76709 75527 75786 78312 77413 76965 76806 79114 
    ##    27    28    29    30    31 
    ## 79789 77059 71712 74931 74849

``` r
summary(met$temp)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##  -40.00   19.60   23.50   23.59   27.80   56.00   60089

``` r
summary(met$elev)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   -13.0   101.0   252.0   415.8   400.0  9999.0

``` r
summary(met$wind.sp)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##    0.00    0.00    2.10    2.46    3.60   36.00   79693

#### Data cleaning

``` r
#met[met$elev==9999.0] <- NA
#met$elev[met$elev==9999.0] <- NA
met[met$elev==9999.0, elev:= NA ]
summary(met$elev)
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##     -13     101     252     413     400    4113     710

At what elevation is the highest weather station?

The weather station with highest elevation is 4113 meters. This is after
replacing 9999.0 values with the appropriate code for “missing”, which
is “NA”.

Remove temps of -40 celsius.

``` r
met <- met[temp>-40]
met2 <- met[order(temp)]
head(met2)
```

    ##    USAFID WBAN year month day hour min    lat    lon elev wind.dir wind.dir.qc
    ## 1: 722817 3068 2019     8   1    0  56 38.767 -104.3 1838      190           5
    ## 2: 722817 3068 2019     8   1    1  56 38.767 -104.3 1838      180           5
    ## 3: 722817 3068 2019     8   3   11  56 38.767 -104.3 1838       NA           9
    ## 4: 722817 3068 2019     8   3   12  56 38.767 -104.3 1838       NA           9
    ## 5: 722817 3068 2019     8   6   21  56 38.767 -104.3 1838      280           5
    ## 6: 722817 3068 2019     8   6   22  56 38.767 -104.3 1838      240           5
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N     7.2          5         NA             9                 9
    ## 2:              N     7.7          5         NA             9                 9
    ## 3:              C     0.0          5         NA             9                 9
    ## 4:              C     0.0          5         NA             9                 9
    ## 5:              N     2.6          5         NA             9                 9
    ## 6:              N     7.7          5         NA             9                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc  temp temp.qc dew.point
    ## 1:        N       NA           9       N          5 -17.2       5        NA
    ## 2:        N       NA           9       N          5 -17.2       5        NA
    ## 3:        N       NA           9       N          5 -17.2       5        NA
    ## 4:        N       NA           9       N          5 -17.2       5        NA
    ## 5:        N       NA           9       N          5 -17.2       5        NA
    ## 6:        N       NA           9       N          5 -17.2       5        NA
    ##    dew.point.qc atm.press atm.press.qc rh
    ## 1:            9        NA            9 NA
    ## 2:            9        NA            9 NA
    ## 3:            9        NA            9 NA
    ## 4:            9        NA            9 NA
    ## 5:            9        NA            9 NA
    ## 6:            9        NA            9 NA

### 5. Check the data against an external data source.

skip a few steps here…

### 6. Compute summary statistics

``` r
met[elev==max(elev,na.rm=TRUE)][, summary(wind.sp)]
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##   0.000   4.100   6.700   7.245   9.800  21.100     168

``` r
met[elev==max(elev,na.rm=TRUE)][, summary(temp)]
```

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    1.00    6.00    8.00    8.13   10.00   15.00

``` r
elev <- met[elev==max(elev,na.rm=TRUE)]
summary(elev[,.(wind.dir,wind.sp)])
```

    ##     wind.dir        wind.sp      
    ##  Min.   : 10.0   Min.   : 0.000  
    ##  1st Qu.:250.0   1st Qu.: 4.100  
    ##  Median :300.0   Median : 6.700  
    ##  Mean   :261.5   Mean   : 7.245  
    ##  3rd Qu.:310.0   3rd Qu.: 9.800  
    ##  Max.   :360.0   Max.   :21.100  
    ##  NA's   :237     NA's   :168

``` r
met[elev==max(elev,na.rm=TRUE), .(
  temp_wind = cor(temp,wind.sp,use="complete"),
  temp_day  =  cor(temp,day,use="complete"),
  wind_day  =  cor(wind.sp,day,use="complete")
)]
```

    ##      temp_wind     temp_day  wind_day
    ## 1: -0.09373843 -0.003857766 0.3643079

### 7. Exploratory graphs

We should look at the distributions of all of the key variables to make
sure there are no remaining issues with the data.

``` r
hist(met$elev, breaks=100)
```

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
hist(met$wind.sp)
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

One thing we should consider for later analyses is to log transform wind
speed and elevation as the are very skewed.

Look at where the weather station with highest elevation is located.

``` r
#leaflet(elev) %>%
  #addProviderTiles('OpenStreetMap') %>% 
  #addCircles(lat=~lat,lng=~lon, opacity=1, #fillOpacity=1, radius=100)
```

The above doesn’t render in .md.

Look at the time series of temperature and wind speed at this location.
For this we will need to create a date-time variable for the x-axis.

``` r
library(lubridate)
```

    ## 
    ## Attaching package: 'lubridate'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     hour, isoweek, mday, minute, month, quarter, second, wday, week,
    ##     yday, year

    ## The following objects are masked from 'package:base':
    ## 
    ##     date, intersect, setdiff, union

``` r
elev$date <- with(elev, ymd_hm(paste(year, month, day, hour, min, sep= ' ')))
summary(elev$date)
```

    ##                  Min.               1st Qu.                Median 
    ## "2019-08-01 00:36:00" "2019-08-08 11:52:00" "2019-08-16 22:49:00" 
    ##                  Mean               3rd Qu.                  Max. 
    ## "2019-08-16 14:44:19" "2019-08-24 11:12:00" "2019-08-31 22:35:00"

``` r
elev <- elev[order(date)]
head(elev)
```

    ##    USAFID WBAN year month day hour min  lat      lon elev wind.dir wind.dir.qc
    ## 1: 720385  419 2019     8   1    0  36 39.8 -105.766 4113      170           5
    ## 2: 720385  419 2019     8   1    0  54 39.8 -105.766 4113      100           5
    ## 3: 720385  419 2019     8   1    1  12 39.8 -105.766 4113       90           5
    ## 4: 720385  419 2019     8   1    1  35 39.8 -105.766 4113      110           5
    ## 5: 720385  419 2019     8   1    1  53 39.8 -105.766 4113      120           5
    ## 6: 720385  419 2019     8   1    2  12 39.8 -105.766 4113      120           5
    ##    wind.type.code wind.sp wind.sp.qc ceiling.ht ceiling.ht.qc ceiling.ht.method
    ## 1:              N     8.8          5       1372             5                 M
    ## 2:              N     2.6          5       1372             5                 M
    ## 3:              N     3.1          5       1981             5                 M
    ## 4:              N     4.1          5       2134             5                 M
    ## 5:              N     4.6          5       2134             5                 M
    ## 6:              N     6.2          5      22000             5                 9
    ##    sky.cond vis.dist vis.dist.qc vis.var vis.var.qc temp temp.qc dew.point
    ## 1:        N       NA           9       N          5    9       5         1
    ## 2:        N       NA           9       N          5    9       5         1
    ## 3:        N       NA           9       N          5    9       5         2
    ## 4:        N       NA           9       N          5    9       5         2
    ## 5:        N       NA           9       N          5    9       5         2
    ## 6:        N       NA           9       N          5    9       5         2
    ##    dew.point.qc atm.press atm.press.qc       rh                date
    ## 1:            5        NA            9 57.61039 2019-08-01 00:36:00
    ## 2:            5        NA            9 57.61039 2019-08-01 00:54:00
    ## 3:            5        NA            9 61.85243 2019-08-01 01:12:00
    ## 4:            5        NA            9 61.85243 2019-08-01 01:35:00
    ## 5:            5        NA            9 61.85243 2019-08-01 01:53:00
    ## 6:            5        NA            9 61.85243 2019-08-01 02:12:00

With the date-time variable we can plot the time series of temperature
and wind speed.

``` r
plot(elev$date, elev$temp, type='l')
```

![](README_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

``` r
#elev[,plot(date,temp,type="l")]
```

``` r
plot(elev$date, elev$wind.sp, type='l')
```

![](README_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

Summarize any trends that you see in these time-series plots.

``` r
elev2 <- elev[!is.na(wind.sp)]
plot(elev2$date, elev2$wind.sp, type='l')
lines(smooth.spline(elev2$date, elev2$wind.sp,
                    nknots=7),col=2,lwd=2)
```

![](README_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

### SessionInfo

``` r
sessionInfo()
```

    ## R version 4.1.0 (2021-05-18)
    ## Platform: x86_64-apple-darwin17.0 (64-bit)
    ## Running under: macOS Mojave 10.14.6
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRblas.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.1/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] lubridate_1.7.10  leaflet_2.0.4.1   forcats_0.5.1     stringr_1.4.0    
    ##  [5] dplyr_1.0.7       purrr_0.3.4       readr_2.0.1       tidyr_1.1.3      
    ##  [9] tibble_3.1.4      ggplot2_3.3.5     tidyverse_1.3.1   data.table_1.14.0
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.7        assertthat_0.2.1  digest_0.6.27     utf8_1.2.2       
    ##  [5] R6_2.5.1          cellranger_1.1.0  backports_1.2.1   reprex_2.0.1     
    ##  [9] evaluate_0.14     httr_1.4.2        highr_0.9         pillar_1.6.2     
    ## [13] rlang_0.4.11      readxl_1.3.1      rstudioapi_0.13   R.utils_2.10.1   
    ## [17] R.oo_1.24.0       rmarkdown_2.10    htmlwidgets_1.5.3 munsell_0.5.0    
    ## [21] broom_0.7.9       compiler_4.1.0    modelr_0.1.8      xfun_0.25        
    ## [25] pkgconfig_2.0.3   htmltools_0.5.2   tidyselect_1.1.1  fansi_0.5.0      
    ## [29] crayon_1.4.1      tzdb_0.1.2        dbplyr_2.1.1      withr_2.4.2      
    ## [33] R.methodsS3_1.8.1 grid_4.1.0        jsonlite_1.7.2    gtable_0.3.0     
    ## [37] lifecycle_1.0.0   DBI_1.1.1         magrittr_2.0.1    scales_1.1.1     
    ## [41] cli_3.0.1         stringi_1.7.4     fs_1.5.0          xml2_1.3.2       
    ## [45] ellipsis_0.3.2    generics_0.1.0    vctrs_0.3.8       tools_4.1.0      
    ## [49] glue_1.4.2        hms_1.1.0         crosstalk_1.1.1   fastmap_1.1.0    
    ## [53] yaml_2.2.1        colorspace_2.0-2  rvest_1.0.1       knitr_1.33       
    ## [57] haven_2.4.3
