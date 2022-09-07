lab 4
================
ks
9/17/2021

## Lab week 4

``` r
library(data.table)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✔ ggplot2 3.3.6     ✔ purrr   0.3.4
    ## ✔ tibble  3.1.8     ✔ dplyr   1.0.9
    ## ✔ tidyr   1.2.0     ✔ stringr 1.4.0
    ## ✔ readr   2.1.2     ✔ forcats 0.5.1

    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::between()   masks data.table::between()
    ## ✖ dplyr::filter()    masks stats::filter()
    ## ✖ dplyr::first()     masks data.table::first()
    ## ✖ dplyr::lag()       masks stats::lag()
    ## ✖ dplyr::last()      masks data.table::last()
    ## ✖ purrr::transpose() masks data.table::transpose()

``` r
library(leaflet)
library(htmlwidgets)
library(webshot)
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
met <- met[temp > -17]

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
), by = "USAFID"]
#Create a region variable for NW, SW, NE, SE based on lon = -98.00 and lat = 39.71 degrees
#Create a categorical variable for elevation as in the lecture slides
met_avg[lat >  39.71 & lon <= -98, region := "Northwest"]
met_avg[lat <= 39.71 & lon <= -98, region := "Southwest"]
met_avg[lat > 39.71  & lon > -98,  region:= "Northeast"]
met_avg[lat <=  39.71&  lon > -98, region := "Southeast"]

met_avg[,table(region)]
```

    ## region
    ## Northeast Northwest Southeast Southwest 
    ##       484       146       649       296

``` r
met_avg[,elev_cat := ifelse(elev>252,"high","low")]
```

## 3. Use geom_violin to examine the wind speed and dew point temperature by region

``` r
ggplot(met_avg, mapping = aes(y = wind.sp, x = 1)) +
 geom_violin() +
  facet_grid(~region)
```

    ## Warning: Removed 15 rows containing non-finite values (stat_ydensity).

![](README_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

## 6. Use stat_summary to examine mean dew point and wind speed by region with standard deviation error bars

Make sure to remove NA

Use fun.data=“mean_sdl” in stat_summary

``` r
p <- met_avg[!is.na(dew.point) ] %>%
  ggplot() + 
    stat_summary(mapping = aes(x = region, y = dew.point),
    fun.data = mean_sdl)
p
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

Add another layer of stats_summary but change the geom to “errorbar”
(see the help).

Describe the graph and what you observe

Dew point temperature is…

Wind speed is…

## 7. Make a map showing the spatial trend in relative h in the US

``` r
#Make sure to remove NA
#Use leaflet()

#Make a colour palette with custom colours
met_avg2 <- met[,.(temp = mean(temp,na.rm=TRUE), lat = mean(lat), lon = mean(lon)),  by=c("USAFID")]
met_avg2 <- met_avg2[!is.na(temp)]
# Generating a color palette
temp.pal <- colorNumeric(c('darkgreen','goldenrod','brown'), domain=met_avg2$temp)
temp.pal
```

    ## function (x) 
    ## {
    ##     if (length(x) == 0 || all(is.na(x))) {
    ##         return(pf(x))
    ##     }
    ##     if (is.null(rng)) 
    ##         rng <- range(x, na.rm = TRUE)
    ##     rescaled <- scales::rescale(x, from = rng)
    ##     if (any(rescaled < 0 | rescaled > 1, na.rm = TRUE)) 
    ##         warning("Some values were outside the color scale and will be treated as NA")
    ##     if (reverse) {
    ##         rescaled <- 1 - rescaled
    ##     }
    ##     pf(rescaled)
    ## }
    ## <bytecode: 0x7fc552b039e0>
    ## <environment: 0x7fc552b02400>
    ## attr(,"colorType")
    ## [1] "numeric"
    ## attr(,"colorArgs")
    ## attr(,"colorArgs")$na.color
    ## [1] "#808080"

``` r
#Use addMarkers to include the top 10 places in relative h (hint: this will be useful rank(-rh) <= 10)
tempmap <- leaflet(met_avg2) %>% 
  # The looks of the Map
  addProviderTiles('CartoDB.Positron') %>% 
  # Some circles
  addCircles(
    lat = ~lat, lng=~lon,
                                                  # HERE IS OUR PAL!
    label = ~paste0(round(temp,2), ' C'), color = ~ temp.pal(temp),
    opacity = 1, fillOpacity = 1, radius = 500
    ) %>%
  # And alegend
  addLegend('bottomleft', pal=temp.pal, values=met_avg2$temp,
          title='Temperature, C', opacity=1)

#tempmap
```

    ## Warning in is.null(x) || is.na(x): 'length(x) = 4 > 1' in coercion to
    ## 'logical(1)'

![](README_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
sessionInfo()
```

    ## R version 4.2.0 (2022-04-22)
    ## Platform: x86_64-apple-darwin17.0 (64-bit)
    ## Running under: macOS Big Sur/Monterey 10.16
    ## 
    ## Matrix products: default
    ## BLAS:   /Library/Frameworks/R.framework/Versions/4.2/Resources/lib/libRblas.0.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/4.2/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ##  [1] webshot_0.5.3     htmlwidgets_1.5.4 leaflet_2.1.1     forcats_0.5.1    
    ##  [5] stringr_1.4.0     dplyr_1.0.9       purrr_0.3.4       readr_2.1.2      
    ##  [9] tidyr_1.2.0       tibble_3.1.8      ggplot2_3.3.6     tidyverse_1.3.1  
    ## [13] data.table_1.14.2
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] fs_1.5.2                lubridate_1.8.0         RColorBrewer_1.1-3     
    ##  [4] httr_1.4.3              tools_4.2.0             backports_1.4.1        
    ##  [7] utf8_1.2.2              R6_2.5.1                rpart_4.1.16           
    ## [10] Hmisc_4.7-0             DBI_1.1.3               colorspace_2.0-3       
    ## [13] nnet_7.3-17             withr_2.5.0             processx_3.6.1         
    ## [16] tidyselect_1.1.2        gridExtra_2.3           compiler_4.2.0         
    ## [19] cli_3.3.0               rvest_1.0.2             htmlTable_2.4.0        
    ## [22] xml2_1.3.3              labeling_0.4.2          scales_1.2.0           
    ## [25] checkmate_2.1.0         callr_3.7.0             digest_0.6.29          
    ## [28] foreign_0.8-82          rmarkdown_2.14          base64enc_0.1-3        
    ## [31] jpeg_0.1-9              pkgconfig_2.0.3         htmltools_0.5.2        
    ## [34] dbplyr_2.2.1            fastmap_1.1.0           highr_0.9              
    ## [37] rlang_1.0.4             readxl_1.4.0            rstudioapi_0.13        
    ## [40] farver_2.1.0            generics_0.1.3          jsonlite_1.8.0         
    ## [43] crosstalk_1.2.0         magrittr_2.0.3          Formula_1.2-4          
    ## [46] interp_1.1-2            Matrix_1.4-1            Rcpp_1.0.8.3           
    ## [49] munsell_0.5.0           fansi_1.0.3             lifecycle_1.0.1        
    ## [52] stringi_1.7.8           yaml_2.3.5              grid_4.2.0             
    ## [55] crayon_1.5.1            deldir_1.0-6            lattice_0.20-45        
    ## [58] haven_2.5.0             splines_4.2.0           hms_1.1.1              
    ## [61] ps_1.7.1                knitr_1.39              pillar_1.8.0           
    ## [64] reprex_2.0.1            glue_1.6.2              evaluate_0.15          
    ## [67] leaflet.providers_1.9.0 latticeExtra_0.6-30     modelr_0.1.8           
    ## [70] png_0.1-7               vctrs_0.4.1             tzdb_0.3.0             
    ## [73] cellranger_1.1.0        gtable_0.3.0            assertthat_0.2.1       
    ## [76] xfun_0.31               broom_1.0.0             survival_3.3-1         
    ## [79] cluster_2.1.3           ellipsis_0.3.2
