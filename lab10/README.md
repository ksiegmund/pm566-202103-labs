lab 10
================
ks
11/05/2021

## Lab week 10

``` r
library(data.table)
library(ggplot2)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:data.table':
    ## 
    ##     between, first, last

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(tidytext)
library(tidyverse)
```

    ## ── Attaching packages ─────────────────────────────────────── tidyverse 1.3.1 ──

    ## ✓ tibble  3.1.4     ✓ purrr   0.3.4
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
library(forcats)

# install.packages(c("RSQLite", "DBI"))

library(RSQLite)
library(DBI)
```

``` r
# Initialize a temporary in memory database
con <- dbConnect(SQLite(), ":memory:")
```

``` r
# Download tables
actor <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/actor.csv")
rental <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/rental.csv")
customer <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/customer.csv")
payment <- read.csv("https://raw.githubusercontent.com/ivanceras/sakila/master/csv-sakila-db/payment_p2007_01.csv")
```

``` r
# Copy data.frames to database
dbWriteTable(con, "actor", actor)
dbWriteTable(con, "rental", rental)
dbWriteTable(con, "customer", customer)
dbWriteTable(con, "payment", payment)
```

``` r
dbListTables(con)
```

    ## [1] "actor"    "customer" "payment"  "rental"

You can also use knitr + SQL!

TIP: Use can use the following QUERY to see the structure of a table

``` sql
PRAGMA table_info(actor)
```

``` r
x1
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

This is equivalent to using `dbGetQuery`.

``` r
dbGetQuery(con, "PRAGMA table_info(actor)")
```

    ##   cid        name    type notnull dflt_value pk
    ## 1   0    actor_id INTEGER       0         NA  0
    ## 2   1  first_name    TEXT       0         NA  0
    ## 3   2   last_name    TEXT       0         NA  0
    ## 4   3 last_update    TEXT       0         NA  0

SQL references:  
<https://www.w3schools.com/sql/>

## Exercise 1

Retrive the actor ID, first name and last name for all actors using the
actor table. Sort by last name and then by first name.

``` r
dbGetQuery(con, " 
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name")
```

    ##     actor_id  first_name    last_name
    ## 1         58   CHRISTIAN       AKROYD
    ## 2        182      DEBBIE       AKROYD
    ## 3         92     KIRSTEN       AKROYD
    ## 4        118        CUBA        ALLEN
    ## 5        145         KIM        ALLEN
    ## 6        194       MERYL        ALLEN
    ## 7         76    ANGELINA      ASTAIRE
    ## 8        112     RUSSELL       BACALL
    ## 9        190      AUDREY       BAILEY
    ## 10        67     JESSICA       BAILEY
    ## 11       115    HARRISON         BALE
    ## 12       187       RENEE         BALL
    ## 13        47       JULIA    BARRYMORE
    ## 14       158      VIVIEN     BASINGER
    ## 15       174     MICHAEL       BENING
    ## 16       124    SCARLETT       BENING
    ## 17        14      VIVIEN       BERGEN
    ## 18       121        LIZA      BERGMAN
    ## 19        91 CHRISTOPHER        BERRY
    ## 20        60       HENRY        BERRY
    ## 21        12        KARL        BERRY
    ## 22       189        CUBA        BIRCH
    ## 23        25       KEVIN        BLOOM
    ## 24       185     MICHAEL       BOLGER
    ## 25        37         VAL       BOLGER
    ## 26        98       CHRIS      BRIDGES
    ## 27        39      GOLDIE        BRODY
    ## 28       159       LAURA        BRODY
    ## 29       167    LAURENCE      BULLOCK
    ## 30        40      JOHNNY         CAGE
    ## 31        11        ZERO         CAGE
    ## 32       181     MATTHEW       CARREY
    ## 33        86        GREG      CHAPLIN
    ## 34         3          ED        CHASE
    ## 35       176         JON        CHASE
    ## 36       183     RUSSELL        CLOSE
    ## 37        16        FRED      COSTNER
    ## 38       129       DARYL     CRAWFORD
    ## 39        26         RIP     CRAWFORD
    ## 40        49        ANNE       CRONYN
    ## 41       104    PENELOPE       CRONYN
    ## 42       105      SIDNEY        CROWE
    ## 43        57        JUDE       CRUISE
    ## 44       201         TOM       CRUISE
    ## 45       203         TOM       CRUISE
    ## 46       205         TOM       CRUISE
    ## 47       207         TOM       CRUISE
    ## 48        80       RALPH         CRUZ
    ## 49        81    SCARLETT        DAMON
    ## 50         4    JENNIFER        DAVIS
    ## 51       101       SUSAN        DAVIS
    ## 52       110       SUSAN        DAVIS
    ## 53        48     FRANCES    DAY-LEWIS
    ## 54        35        JUDY         DEAN
    ## 55       143       RIVER         DEAN
    ## 56       148       EMILY          DEE
    ## 57       138     LUCILLE          DEE
    ## 58       107        GINA    DEGENERES
    ## 59        41       JODIE    DEGENERES
    ## 60       166        NICK    DEGENERES
    ## 61        89    CHARLIZE        DENCH
    ## 62       123    JULIANNE        DENCH
    ## 63       160       CHRIS         DEPP
    ## 64       100     SPENCER         DEPP
    ## 65       109   SYLVESTER         DERN
    ## 66       173        ALAN     DREYFUSS
    ## 67        36        BURT      DUKAKIS
    ## 68       188        ROCK      DUKAKIS
    ## 69       106     GROUCHO        DUNST
    ## 70        19         BOB      FAWCETT
    ## 71       199       JULIA      FAWCETT
    ## 72        10   CHRISTIAN        GABLE
    ## 73       165          AL      GARLAND
    ## 74       184    HUMPHREY      GARLAND
    ## 75       127       KEVIN      GARLAND
    ## 76       154       MERYL       GIBSON
    ## 77        46      PARKER     GOLDBERG
    ## 78       139        EWAN      GOODING
    ## 79       191     GREGORY      GOODING
    ## 80        71        ADAM        GRANT
    ## 81       179          ED      GUINESS
    ## 82         1    PENELOPE      GUINESS
    ## 83        90        SEAN      GUINESS
    ## 84        32         TIM      HACKMAN
    ## 85       175     WILLIAM      HACKMAN
    ## 86       202         TOM        HANKS
    ## 87       204         TOM        HANKS
    ## 88       206         TOM        HANKS
    ## 89       208         TOM        HANKS
    ## 90       152         BEN       HARRIS
    ## 91       141        CATE       HARRIS
    ## 92        56         DAN       HARRIS
    ## 93        97         MEG        HAWKE
    ## 94       151    GEOFFREY       HESTON
    ## 95       169     KENNETH      HOFFMAN
    ## 96        79         MAE      HOFFMAN
    ## 97        28       WOODY      HOFFMAN
    ## 98       161      HARVEY         HOPE
    ## 99       134        GENE      HOPKINS
    ## 100      113      MORGAN      HOPKINS
    ## 101       50     NATALIE      HOPKINS
    ## 102      132        ADAM       HOPPER
    ## 103      170        MENA       HOPPER
    ## 104       65      ANGELA       HUDSON
    ## 105       52      CARMEN         HUNT
    ## 106      140      WHOOPI         HURT
    ## 107      131        JANE      JACKMAN
    ## 108      119      WARREN      JACKMAN
    ## 109      146      ALBERT    JOHANSSON
    ## 110        8     MATTHEW    JOHANSSON
    ## 111       64         RAY    JOHANSSON
    ## 112       82       WOODY        JOLIE
    ## 113       43        KIRK     JOVOVICH
    ## 114      130       GRETA       KEITEL
    ## 115      198        MARY       KEITEL
    ## 116       74       MILLA       KEITEL
    ## 117       55         FAY       KILMER
    ## 118      153      MINNIE       KILMER
    ## 119      162       OPRAH       KILMER
    ## 120       45       REESE       KILMER
    ## 121       23      SANDRA       KILMER
    ## 122      103     MATTHEW        LEIGH
    ## 123        5      JOHNNY LOLLOBRIGIDA
    ## 124      157       GRETA       MALDEN
    ## 125      136          ED    MANSFIELD
    ## 126       22       ELVIS         MARX
    ## 127       77        CARY  MCCONAUGHEY
    ## 128       70    MICHELLE  MCCONAUGHEY
    ## 129      114      MORGAN    MCDORMAND
    ## 130      177        GENE     MCKELLEN
    ## 131       38         TOM     MCKELLEN
    ## 132      128        CATE      MCQUEEN
    ## 133       27       JULIA      MCQUEEN
    ## 134       42         TOM      MIRANDA
    ## 135      178        LISA       MONROE
    ## 136      120    PENELOPE       MONROE
    ## 137        7       GRACE       MOSTEL
    ## 138       99         JIM       MOSTEL
    ## 139       61   CHRISTIAN       NEESON
    ## 140       62       JAYNE       NEESON
    ## 141        6       BETTE    NICHOLSON
    ## 142      125      ALBERT        NOLTE
    ## 143      150       JAYNE        NOLTE
    ## 144      122       SALMA        NOLTE
    ## 145      108      WARREN        NOLTE
    ## 146       34      AUDREY      OLIVIER
    ## 147       15        CUBA      OLIVIER
    ## 148       69     KENNETH      PALTROW
    ## 149       21     KIRSTEN      PALTROW
    ## 150       33       MILLA         PECK
    ## 151       30      SANDRA         PECK
    ## 152       87     SPENCER         PECK
    ## 153       73        GARY         PENN
    ## 154      133     RICHARD         PENN
    ## 155       88     KENNETH        PESCI
    ## 156      171     OLYMPIA     PFEIFFER
    ## 157       51        GARY      PHOENIX
    ## 158       54    PENELOPE      PINKETT
    ## 159       84       JAMES         PITT
    ## 160       75        BURT        POSEY
    ## 161       93       ELLEN      PRESLEY
    ## 162      135        RITA     REYNOLDS
    ## 163      142        JADA        RYDER
    ## 164      195       JAYNE  SILVERSTONE
    ## 165      180        JEFF  SILVERSTONE
    ## 166       78     GROUCHO      SINATRA
    ## 167       31       SISSY     SOBIESKI
    ## 168       44        NICK     STALLONE
    ## 169       24     CAMERON       STREEP
    ## 170      116         DAN       STREEP
    ## 171      192        JOHN       SUVARI
    ## 172        9         JOE        SWANK
    ## 173      155         IAN        TANDY
    ## 174       66        MARY        TANDY
    ## 175       59      DUSTIN       TAUTOU
    ## 176      193        BURT       TEMPLE
    ## 177       53        MENA       TEMPLE
    ## 178      149     RUSSELL       TEMPLE
    ## 179      200       THORA       TEMPLE
    ## 180      126     FRANCES        TOMEI
    ## 181       18         DAN         TORN
    ## 182       94     KENNETH         TORN
    ## 183      102      WALTER         TORN
    ## 184       20     LUCILLE        TRACY
    ## 185      117       RENEE        TRACY
    ## 186       17       HELEN       VOIGHT
    ## 187       95       DARYL     WAHLBERG
    ## 188        2        NICK     WAHLBERG
    ## 189      196        BELA       WALKEN
    ## 190       29        ALEC        WAYNE
    ## 191      163 CHRISTOPHER         WEST
    ## 192      197       REESE         WEST
    ## 193      172     GROUCHO     WILLIAMS
    ## 194      137      MORGAN     WILLIAMS
    ## 195       72        SEAN     WILLIAMS
    ## 196       83         BEN       WILLIS
    ## 197       96        GENE       WILLIS
    ## 198      164    HUMPHREY       WILLIS
    ## 199      168        WILL       WILSON
    ## 200      147         FAY      WINSLET
    ## 201       68         RIP      WINSLET
    ## 202      144      ANGELA  WITHERSPOON
    ## 203      156         FAY         WOOD
    ## 204       13         UMA         WOOD
    ## 205       63     CAMERON         WRAY
    ## 206      111     CAMERON    ZELLWEGER
    ## 207      186       JULIA    ZELLWEGER
    ## 208       85      MINNIE    ZELLWEGER

## Exercise 2

Retrive the actor ID, first name, and last name for actors whose last
name equals ‘WILLIAMS’ or ‘DAVIS’.

SELECT FROM WHERE \_\_\_ IN (‘WILLIAMS’, ‘DAVIS’) Exercise 3 Write a
query against the rental table that returns the IDs of the customers who
rented a film on July 5, 2005 (use the rental.rental\_date column, and
you can use the date() function to ignore the time component). Include a
single row for each distinct customer ID.

SELECT DISTINCT FROM WHERE date(\_\_\_) = ‘2005-07-05’ Exercise 4
Exercise 4.1 Construct a query that retrives all rows from the payment
table where the amount is either 1.99, 7.99, 9.99.

SELECT \* FROM *** WHERE *** IN (1.99, 7.99, 9.99) Exercise 4.2
Construct a query that retrives all rows from the payment table where
the amount is greater then 5

SELECT \* FROM WHERE Exercise 4.2 Construct a query that retrives all
rows from the payment table where the amount is greater then 5 and less
then 8

SELECT \* FROM *** WHERE *** AND \_\_\_ Exercise 5 Retrive all the
payment IDs and their amount from the customers whose last name is
‘DAVIS’.

SELECT FROM INNER JOIN WHERE AND Exercise 6 Exercise 6.1 Use COUNT(\*)
to count the number of rows in rental

Exercise 6.2 Use COUNT(\*) and GROUP BY to count the number of rentals
for each customer\_id

Exercise 6.3 Repeat the previous query and sort by the count in
descending order

Exercise 6.4 Repeat the previous query but use HAVING to only keep the
groups with 40 or more.

Exercise 7 The following query calculates a number of summary statistics
for the payment table using MAX, MIN, AVG and SUM

Exercise 7.1 Modify the above query to do those calculations for each
customer\_id

Exercise 7.2 Modify the above query to only keep the customer\_ids that
have more then 5 payments

Cleanup Run the following chunk to disconnect from the connection.

``` r
dbDisconnect(con)
```

Knit the document, commit your changes, and Save it on GitHub.

git commit -a -m “Finalizing lab 10
<https://github.com/USCbiostats/PM566/issues/47>”

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
    ##  [1] DBI_1.1.1         RSQLite_2.2.8     forcats_0.5.1     stringr_1.4.0    
    ##  [5] purrr_0.3.4       readr_2.0.1       tidyr_1.1.3       tibble_3.1.4     
    ##  [9] tidyverse_1.3.1   tidytext_0.3.2    dplyr_1.0.7       ggplot2_3.3.5    
    ## [13] data.table_1.14.0
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.7        lubridate_1.7.10  lattice_0.20-44   assertthat_0.2.1 
    ##  [5] digest_0.6.27     utf8_1.2.2        R6_2.5.1          cellranger_1.1.0 
    ##  [9] backports_1.2.1   reprex_2.0.1      evaluate_0.14     httr_1.4.2       
    ## [13] pillar_1.6.2      rlang_0.4.11      readxl_1.3.1      rstudioapi_0.13  
    ## [17] blob_1.2.2        Matrix_1.3-4      rmarkdown_2.10    bit_4.0.4        
    ## [21] munsell_0.5.0     broom_0.7.9       compiler_4.1.0    janeaustenr_0.1.5
    ## [25] modelr_0.1.8      xfun_0.25         pkgconfig_2.0.3   htmltools_0.5.2  
    ## [29] tidyselect_1.1.1  fansi_0.5.0       crayon_1.4.1      tzdb_0.1.2       
    ## [33] dbplyr_2.1.1      withr_2.4.2       SnowballC_0.7.0   grid_4.1.0       
    ## [37] jsonlite_1.7.2    gtable_0.3.0      lifecycle_1.0.0   magrittr_2.0.1   
    ## [41] scales_1.1.1      tokenizers_0.2.1  cachem_1.0.6      cli_3.0.1        
    ## [45] stringi_1.7.4     fs_1.5.0          xml2_1.3.2        ellipsis_0.3.2   
    ## [49] generics_0.1.0    vctrs_0.3.8       tools_4.1.0       bit64_4.0.5      
    ## [53] glue_1.4.2        hms_1.1.0         fastmap_1.1.0     yaml_2.2.1       
    ## [57] colorspace_2.0-2  rvest_1.0.1       memoise_2.0.0     knitr_1.34       
    ## [61] haven_2.4.3
