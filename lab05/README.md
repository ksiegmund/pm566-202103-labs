lab 5
================
ks
9/24/2021

## Lab week 5

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

### 1. Read in the met data

First read the data into a data.table.

``` r
if (!file.exists("~kims/GitHub/pm566-202103-labs/lab03/met_all.gz")) {
download.file("https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz", "met_all.gz", method="libcurl", timeout = 60)
}
dat <- data.table::fread("~kims/GitHub/pm566-202103-labs/lab03/met_all.gz")
```

### 2. Read in the stations data

``` r
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

    ## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion

``` r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates (quick and dirty, not recommended)
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```

### 3. Merge the two data tables

``` r
dat <- merge(
  # Data
  x     = dat,      
  y     = stations, 
  # List of variables to match
  by.x  = "USAFID",
  by.y  = "USAF", 
  # Which obs to keep?
  all.x = TRUE,      
  all.y = FALSE
  )
head(dat[, list(USAFID, WBAN, STATE)], n = 4)
```

    ##    USAFID  WBAN STATE
    ## 1: 690150 93121    CA
    ## 2: 690150 93121    CA
    ## 3: 690150 93121    CA
    ## 4: 690150 93121    CA

### Question 1: Representative station for the US

What is the median station in terms of temperature, wind speed, and
atmospheric pressure? Look for the three weather stations that best
represent continental US using the quantile() function. Do these three
coincide?

First, generate a representative version of each station. We will use
averages (could use medians too).

``` r
station_averages <- dat[,.(
  temp      = mean(temp,na.rm = TRUE),
  wind.sp   = mean(wind.sp,na.rm = TRUE),
  atm.press = mean(temp,na.rm = TRUE)
), by = USAFID]
```

Now we want to find quantiles per variable.

``` r
medians <- station_averages[,.(
  temp_50       = quantile(temp,      probs = 0.5, na.rm = TRUE),
  wind.sp_50    = quantile(wind.sp,   probs = 0.5, na.rm = TRUE),
  atm.press_50  = quantile(atm.press, probs = 0.5, na.rm = TRUE)
)]
```

Now we can find the stations that are closest to these. (hint: use the
function ‘which.min()’)

``` r
station_averages[, temp_dist := abs(temp- medians$temp_50)]
station_averages[order(temp_dist)][1]
```

    ##    USAFID     temp  wind.sp atm.press   temp_dist
    ## 1: 720458 23.68173 1.209682  23.68173 0.002328907

Knit the document, commit your changes, and Save it on GitHub. Don’t
forget to add README.md to the tree, the first time you render it.

### Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the
most representative, the median, station per state. This time, instead
of looking at one variable at a time, look at the euclidean distance. If
multiple stations show in the median, select the one located at the
lowest latitude.

``` r
station_averages <- dat[,.(
  temp      = mean(temp,na.rm = TRUE),
  wind.sp   = mean(wind.sp,na.rm = TRUE),
  atm.press = mean(temp,na.rm = TRUE)
), by = .(USAFID,STATE)]
```

``` r
station_averages[, temp_50 := quantile(temp,probs = 0.5, na.rm = TRUE), by = STATE]
station_averages[, wind.sp_50 := quantile(wind.sp,probs = 0.5, na.rm = TRUE), by = STATE]  
station_averages[, atm.press_50 := quantile(atm.press,probs = 0.5, na.rm = TRUE), by = STATE]
head(station_averages)
```

    ##    USAFID STATE     temp  wind.sp atm.press  temp_50 wind.sp_50 atm.press_50
    ## 1: 690150    CA 33.18763 3.483560  33.18763 22.66268   2.565445     22.66268
    ## 2: 720110    TX 31.22003 2.138348  31.22003 29.75188   3.413737     29.75188
    ## 3: 720113    MI 23.29317 2.470298  23.29317 20.51970   2.273423     20.51970
    ## 4: 720120    SC 27.01922 2.504692  27.01922 25.80545   1.696119     25.80545
    ## 5: 720137    IL 21.88823 1.979335  21.88823 22.43194   2.237622     22.43194
    ## 6: 720151    TX 27.57686 2.998428  27.57686 29.75188   3.413737     29.75188

``` r
station_averages[, eucldist := sqrt(
   (temp - temp_50)^2 + (wind.sp - wind.sp_50)^2
)]
station_averages
```

    ##       USAFID STATE     temp  wind.sp atm.press  temp_50 wind.sp_50 atm.press_50
    ##    1: 690150    CA 33.18763 3.483560  33.18763 22.66268   2.565445     22.66268
    ##    2: 720110    TX 31.22003 2.138348  31.22003 29.75188   3.413737     29.75188
    ##    3: 720113    MI 23.29317 2.470298  23.29317 20.51970   2.273423     20.51970
    ##    4: 720120    SC 27.01922 2.504692  27.01922 25.80545   1.696119     25.80545
    ##    5: 720137    IL 21.88823 1.979335  21.88823 22.43194   2.237622     22.43194
    ##   ---                                                                          
    ## 1591: 726777    MT 19.15492 4.673878  19.15492 19.15492   4.151737     19.15492
    ## 1592: 726797    MT 18.78980 2.858586  18.78980 19.15492   4.151737     19.15492
    ## 1593: 726798    MT 19.47014 4.445783  19.47014 19.15492   4.151737     19.15492
    ## 1594: 726810    ID 25.03549 3.039794  25.03549 20.56798   2.568944     20.56798
    ## 1595: 726813    ID 23.47809 2.435372  23.47809 20.56798   2.568944     20.56798
    ##         eucldist
    ##    1: 10.5649277
    ##    2:  1.9447578
    ##    3:  2.7804480
    ##    4:  1.4584280
    ##    5:  0.6019431
    ##   ---           
    ## 1591:  0.5221409
    ## 1592:  1.3437090
    ## 1593:  0.4310791
    ## 1594:  4.4922623
    ## 1595:  2.9131751

Knit the doc and save it on GitHub.

### Question 3: In the middle?

For each state, identify what is the station that is closest to the
mid-point of the state. Combining these with the stations you identified
in the previous question, use leaflet() to visualize all \~100 points in
the same figure, applying different colors for those identified in this
question.

Knit the doc and save it on GitHub.

### Question 4: Means of means

Using the quantile() function, generate a summary table that shows the
number of states included, average temperature, wind-speed, and
atmospheric pressure by the variable “average temperature level,” which
you’ll need to create.

Start by computing the states’ average temperature. Use that measurement
to classify them according to the following criteria:

low: temp &lt; 20 Mid: temp &gt;= 20 and temp &lt; 25 High: temp &gt;=
25 Once you are done with that, you can compute the following:

Number of entries (records), Number of NA entries, Number of stations,
Number of states included, and Mean temperature, wind-speed, and
atmospheric pressure. All by the levels described before.

Knit the document, commit your changes, and push them to GitHub. If
you’d like, you can take this time to include the link of the issue of
the week so that you let us know when you are done, e.g.,

git commit -a -m “Finalizing lab 5
<https://github.com/USCbiostats/PM566/issues/23>”

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
    ##  [1] leaflet_2.0.4.1   forcats_0.5.1     stringr_1.4.0     dplyr_1.0.7      
    ##  [5] purrr_0.3.4       readr_2.0.1       tidyr_1.1.3       tibble_3.1.4     
    ##  [9] ggplot2_3.3.5     tidyverse_1.3.1   data.table_1.14.0
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] tidyselect_1.1.1  xfun_0.25         haven_2.4.3       colorspace_2.0-2 
    ##  [5] vctrs_0.3.8       generics_0.1.0    htmltools_0.5.2   yaml_2.2.1       
    ##  [9] utf8_1.2.2        rlang_0.4.11      pillar_1.6.2      glue_1.4.2       
    ## [13] withr_2.4.2       DBI_1.1.1         dbplyr_2.1.1      modelr_0.1.8     
    ## [17] readxl_1.3.1      lifecycle_1.0.0   munsell_0.5.0     gtable_0.3.0     
    ## [21] cellranger_1.1.0  rvest_1.0.1       htmlwidgets_1.5.4 evaluate_0.14    
    ## [25] knitr_1.34        tzdb_0.1.2        fastmap_1.1.0     crosstalk_1.1.1  
    ## [29] fansi_0.5.0       broom_0.7.9       Rcpp_1.0.7        scales_1.1.1     
    ## [33] backports_1.2.1   jsonlite_1.7.2    fs_1.5.0          hms_1.1.0        
    ## [37] digest_0.6.27     stringi_1.7.4     grid_4.1.0        cli_3.0.1        
    ## [41] tools_4.1.0       magrittr_2.0.1    crayon_1.4.1      pkgconfig_2.0.3  
    ## [45] ellipsis_0.3.2    xml2_1.3.2        reprex_2.0.1      lubridate_1.7.10 
    ## [49] rstudioapi_0.13   assertthat_0.2.1  rmarkdown_2.10    httr_1.4.2       
    ## [53] R6_2.5.1          compiler_4.1.0