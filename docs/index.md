lab 7
================
ks
10/08/2021

## Lab week 7

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
fn <- "~kims/GitHub/pm566-202103-labs/lab06/mtsamples.csv"
if (!file.exists(fn))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/00_mtsamples/mtsamples.csv",
    destfile = fn)
mtsamples <- read_csv(fn)
```

    ## New names:
    ## * `` -> ...1

    ## Rows: 4999 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (5): description, medical_specialty, sample_name, transcription, keywords
    ## dbl (1): ...1

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#mtsamples <- as_tibble(fn)
```

Read in Medical Transcriptions Loading in reference transcription
samples from <https://www.mtsamples.com/>

``` r
dim(mtsamples)
```

    ## [1] 4999    6

``` r
colnames(mtsamples)
```

    ## [1] "...1"              "description"       "medical_specialty"
    ## [4] "sample_name"       "transcription"     "keywords"

Knit the document, commit your changes, and Save it on GitHub.

git commit -a -m “Finalizing lab 6
<https://github.com/USCbiostats/PM566/issues/43>”

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
    ##  [1] forcats_0.5.1     stringr_1.4.0     dplyr_1.0.7       purrr_0.3.4      
    ##  [5] readr_2.0.1       tidyr_1.1.3       tibble_3.1.4      ggplot2_3.3.5    
    ##  [9] tidyverse_1.3.1   data.table_1.14.0
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] tidyselect_1.1.1 xfun_0.25        haven_2.4.3      colorspace_2.0-2
    ##  [5] vctrs_0.3.8      generics_0.1.0   htmltools_0.5.2  yaml_2.2.1      
    ##  [9] utf8_1.2.2       rlang_0.4.11     pillar_1.6.2     glue_1.4.2      
    ## [13] withr_2.4.2      DBI_1.1.1        bit64_4.0.5      dbplyr_2.1.1    
    ## [17] modelr_0.1.8     readxl_1.3.1     lifecycle_1.0.0  munsell_0.5.0   
    ## [21] gtable_0.3.0     cellranger_1.1.0 rvest_1.0.1      codetools_0.2-18
    ## [25] evaluate_0.14    knitr_1.34       tzdb_0.1.2       fastmap_1.1.0   
    ## [29] parallel_4.1.0   fansi_0.5.0      broom_0.7.9      Rcpp_1.0.7      
    ## [33] scales_1.1.1     backports_1.2.1  vroom_1.5.4      jsonlite_1.7.2  
    ## [37] bit_4.0.4        fs_1.5.0         hms_1.1.0        digest_0.6.27   
    ## [41] stringi_1.7.4    grid_4.1.0       cli_3.0.1        tools_4.1.0     
    ## [45] magrittr_2.0.1   crayon_1.4.1     pkgconfig_2.0.3  ellipsis_0.3.2  
    ## [49] xml2_1.3.2       reprex_2.0.1     lubridate_1.7.10 rstudioapi_0.13 
    ## [53] assertthat_0.2.1 rmarkdown_2.10   httr_1.4.2       R6_2.5.1        
    ## [57] compiler_4.1.0
