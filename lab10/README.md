Lab 10
================
ks
11/05/2021

## Lab week 10

``` r
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

TIP: Use can use the following QUERY to see the structure of a table.

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

Using LIMIT n, we can print just the first n entries.

``` r
dbGetQuery(con, " 
SELECT actor_id, first_name, last_name
FROM actor
ORDER by last_name, first_name
LIMIT 6")
```

    ##   actor_id first_name last_name
    ## 1       58  CHRISTIAN    AKROYD
    ## 2      182     DEBBIE    AKROYD
    ## 3       92    KIRSTEN    AKROYD
    ## 4      118       CUBA     ALLEN
    ## 5      145        KIM     ALLEN
    ## 6      194      MERYL     ALLEN

## Exercise 2

Retrive the actor ID, first name, and last name for actors whose last
name equals ‘WILLIAMS’ or ‘DAVIS’.

``` r
dbGetQuery(con, " 
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name IN ('WILLIAMS', 'DAVIS')
")
```

    ##   actor_id first_name last_name
    ## 1        4   JENNIFER     DAVIS
    ## 2       72       SEAN  WILLIAMS
    ## 3      101      SUSAN     DAVIS
    ## 4      110      SUSAN     DAVIS
    ## 5      137     MORGAN  WILLIAMS
    ## 6      172    GROUCHO  WILLIAMS

This also works:

``` r
dbGetQuery(con, " 
SELECT actor_id, first_name, last_name
FROM actor
WHERE last_name='WILLIAMS' OR last_name='DAVIS'
")
```

    ##   actor_id first_name last_name
    ## 1        4   JENNIFER     DAVIS
    ## 2       72       SEAN  WILLIAMS
    ## 3      101      SUSAN     DAVIS
    ## 4      110      SUSAN     DAVIS
    ## 5      137     MORGAN  WILLIAMS
    ## 6      172    GROUCHO  WILLIAMS

## Exercise 3

Write a query against the rental table that returns the IDs of the
customers who rented a film on July 5, 2005 (use the rental.rental\_date
column, and you can use the date() function to ignore the time
component). Include a single row for each distinct customer ID.

``` r
dbGetQuery(con, "PRAGMA table_info(rental)")
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0    rental_id INTEGER       0         NA  0
    ## 2   1  rental_date    TEXT       0         NA  0
    ## 3   2 inventory_id INTEGER       0         NA  0
    ## 4   3  customer_id INTEGER       0         NA  0
    ## 5   4  return_date    TEXT       0         NA  0
    ## 6   5     staff_id INTEGER       0         NA  0
    ## 7   6  last_update    TEXT       0         NA  0

``` r
dbGetQuery(con, " 
SELECT DISTINCT customer_id 
FROM  rental
WHERE date(rental_date) = '2005-07-05'
LIMIT 6")
```

    ##   customer_id
    ## 1         565
    ## 2         242
    ## 3          37
    ## 4          60
    ## 5         594
    ## 6           8

## Exercise 4

### Exercise 4.1

Construct a query that retrieves all rows from the payment table where
the amount is either 1.99, 7.99, 9.99.

``` r
dbGetQuery(con, "PRAGMA table_info(payment)")
```

    ##   cid         name    type notnull dflt_value pk
    ## 1   0   payment_id INTEGER       0         NA  0
    ## 2   1  customer_id INTEGER       0         NA  0
    ## 3   2     staff_id INTEGER       0         NA  0
    ## 4   3    rental_id INTEGER       0         NA  0
    ## 5   4       amount    REAL       0         NA  0
    ## 6   5 payment_date    TEXT       0         NA  0

If you have a really big dataset you may want to do it this way:

``` r
qq<- dbSendQuery(con, " 
SELECT *
FROM payment
WHERE amount IN (1.99, 7.99, 9.99)
")
dbFetch(qq,n=10)
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16050         269        2         7   1.99 2007-01-24 21:40:19.996577
    ## 2       16056         270        1       193   1.99 2007-01-26 05:10:14.996577
    ## 3       16081         282        2        48   1.99 2007-01-25 04:49:12.996577
    ## 4       16103         294        1       595   1.99 2007-01-28 12:28:20.996577
    ## 5       16133         307        1       614   1.99 2007-01-28 14:01:54.996577
    ## 6       16158         316        1      1065   1.99 2007-01-31 07:23:22.996577
    ## 7       16160         318        1       224   9.99 2007-01-26 08:46:53.996577
    ## 8       16161         319        1        15   9.99 2007-01-24 23:07:48.996577
    ## 9       16180         330        2       967   7.99 2007-01-30 17:40:32.996577
    ## 10      16206         351        1      1137   1.99 2007-01-31 17:48:40.996577

Now we can grab the next 10 and then close.

``` r
dbFetch(qq,n=10)
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16210         354        2       158   1.99 2007-01-25 23:55:37.996577
    ## 2       16240         369        2       913   7.99 2007-01-30 09:33:24.996577
    ## 3       16275         386        1       583   7.99 2007-01-28 10:17:21.996577
    ## 4       16277         387        1       697   7.99 2007-01-29 00:32:30.996577
    ## 5       16289         391        1       891   7.99 2007-01-30 06:11:38.996577
    ## 6       16302         400        2       516   1.99 2007-01-28 01:40:13.996577
    ## 7       16306         401        2       811   1.99 2007-01-29 17:59:08.996577
    ## 8       16307         402        2       801   1.99 2007-01-29 16:04:16.996577
    ## 9       16314         407        1       619   7.99 2007-01-28 14:20:52.996577
    ## 10      16320         411        2       972   1.99 2007-01-30 18:49:33.996577

``` r
dbClearResult(qq)
```

### Exercise 4.2

Construct a query that retrieves all rows from the payment table where
the amount is greater than 5.

``` r
dbGetQuery(con, " 
SELECT *
FROM payment
WHERE amount > 5
LIMIT 10
")
```

    ##    payment_id customer_id staff_id rental_id amount               payment_date
    ## 1       16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2       16058         271        1      1096   8.99 2007-01-31 11:59:15.996577
    ## 3       16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 4       16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 5       16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 6       16073         276        1       860  10.99 2007-01-30 01:13:42.996577
    ## 7       16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 8       16082         282        2       282   6.99 2007-01-26 17:24:52.996577
    ## 9       16086         284        1      1145   6.99 2007-01-31 18:42:11.996577
    ## 10      16087         286        2        81   6.99 2007-01-25 10:43:45.996577

``` r
dbGetQuery(con, " 
SELECT staff_id, COUNT(*)
FROM payment
/* GROUP BY goes AFTER WHERE*/
WHERE amount > 5
GROUP BY staff_id
")
```

    ##   staff_id COUNT(*)
    ## 1        1      151
    ## 2        2      115

### Exercise 4.3

Construct a query that retrieves all rows from the payment table where
the amount is greater then 5 and less then 8.

``` r
dbGetQuery(con, " 
SELECT *
FROM payment
WHERE amount > 5 AND amount < 8
LIMIT 6")
```

    ##   payment_id customer_id staff_id rental_id amount               payment_date
    ## 1      16052         269        2       678   6.99 2007-01-28 21:44:14.996577
    ## 2      16060         272        1       405   6.99 2007-01-27 12:01:05.996577
    ## 3      16061         272        1      1041   6.99 2007-01-31 04:14:49.996577
    ## 4      16068         274        1       394   5.99 2007-01-27 09:54:37.996577
    ## 5      16074         277        2       308   6.99 2007-01-26 20:30:05.996577
    ## 6      16082         282        2       282   6.99 2007-01-26 17:24:52.996577

## Exercise 5

Retrieve all the payment IDs and their amount from the customers whose
last name is ‘DAVIS’.

``` r
dbGetQuery(con, " 
SELECT last_name, payment_id, amount 
FROM customer AS a INNER JOIN payment AS b 
  ON a.customer_id = b.customer_id
WHERE last_name IS 'DAVIS'
")
```

    ##   last_name payment_id amount
    ## 1     DAVIS      16685   4.99
    ## 2     DAVIS      16686   2.99
    ## 3     DAVIS      16687   0.99

## Exercise 6

### Exercise 6.1

Use COUNT(\*) to count the number of rows in rental.

``` r
dbGetQuery(con,"
 SELECT COUNT(*)
 FROM rental
")
```

    ##   COUNT(*)
    ## 1    16044

### Exercise 6.2

Use COUNT(\*) and GROUP BY to count the number of rentals for each
customer\_id.

``` r
dbGetQuery(con,"
 SELECT customer_id, COUNT(*) AS  `N Rentals`
 FROM rental
 GROUP BY customer_id  LIMIT 6
")
```

    ##   customer_id N Rentals
    ## 1           1        32
    ## 2           2        27
    ## 3           3        26
    ## 4           4        22
    ## 5           5        38
    ## 6           6        28

### Exercise 6.3

Repeat the previous query and sort by the count in descending order.

``` r
dbGetQuery(con,"
 SELECT customer_id, COUNT(*) AS count
 FROM rental
 GROUP BY customer_id
 ORDER BY count DESC
 LIMIT 6
 ")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41
    ## 6         469    40

### Exercise 6.4

Repeat the previous query but use HAVING to only keep the groups with 40
or more.

``` r
dbGetQuery(con,"
 SELECT customer_id, COUNT(*) AS count
 FROM rental
 GROUP BY customer_id
  HAVING count > 40
  ORDER BY count DESC
")
```

    ##   customer_id count
    ## 1         148    46
    ## 2         526    45
    ## 3         236    42
    ## 4         144    42
    ## 5          75    41

## Exercise 7

The following query calculates a number of summary statistics for the
payment table using MAX, MIN, AVG and SUM.

``` r
dbGetQuery(con,"
 SELECT MAX(amount) AS max,
        MIN(amount) as min,
        AVG(amount) AS avg,
        SUM(amount) AS sum
 FROM payment
")
```

    ##     max  min      avg     sum
    ## 1 11.99 0.99 4.169775 4824.43

### Exercise 7.1

Modify the above query to do those calculations for each customer\_id.

``` r
dbGetQuery(con,"
 SELECT customer_id, 
        MIN(amount) as min,
        AVG(amount) AS avg,
        MAX(amount) AS max,
        SUM(amount) AS sum
 FROM payment
 GROUP BY customer_id
 LIMIT 10
")
```

    ##    customer_id  min      avg  max   sum
    ## 1            1 0.99 1.990000 2.99  3.98
    ## 2            2 4.99 4.990000 4.99  4.99
    ## 3            3 1.99 2.490000 2.99  4.98
    ## 4            5 0.99 3.323333 6.99  9.97
    ## 5            6 0.99 2.990000 4.99  8.97
    ## 6            7 0.99 4.190000 5.99 20.95
    ## 7            8 6.99 6.990000 6.99  6.99
    ## 8            9 0.99 3.656667 4.99 10.97
    ## 9           10 4.99 4.990000 4.99  4.99
    ## 10          11 6.99 6.990000 6.99  6.99

### Exercise 7.2

Modify the above query to only keep the customer\_ids that have more
then 5 payments.

``` r
dbGetQuery(con,"
 SELECT customer_id, COUNT(*) AS count,
        MIN(amount) as min,
        AVG(amount) AS avg,
        MAX(amount) AS max,
        SUM(amount) AS sum
 FROM payment
 GROUP BY customer_id
 HAVING count  >   5
")
```

    ##    customer_id count  min      avg  max   sum
    ## 1           19     6 0.99 4.490000 9.99 26.94
    ## 2           53     6 0.99 4.490000 9.99 26.94
    ## 3          109     7 0.99 3.990000 7.99 27.93
    ## 4          161     6 0.99 2.990000 5.99 17.94
    ## 5          197     8 0.99 2.615000 3.99 20.92
    ## 6          207     6 0.99 2.990000 6.99 17.94
    ## 7          239     6 2.99 5.656667 7.99 33.94
    ## 8          245     6 0.99 4.823333 8.99 28.94
    ## 9          251     6 1.99 3.323333 4.99 19.94
    ## 10         269     6 0.99 3.156667 6.99 18.94
    ## 11         274     6 2.99 4.156667 5.99 24.94
    ## 12         371     6 0.99 4.323333 6.99 25.94
    ## 13         506     7 0.99 4.132857 8.99 28.93
    ## 14         596     6 0.99 3.823333 6.99 22.94

Cleanup: Run the following chunk to disconnect from the connection.

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
    ## [1] DBI_1.1.1     RSQLite_2.2.8
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_1.0.7      digest_0.6.27   magrittr_2.0.1  evaluate_0.14  
    ##  [5] cachem_1.0.6    rlang_0.4.11    stringi_1.7.4   blob_1.2.2     
    ##  [9] vctrs_0.3.8     rmarkdown_2.10  tools_4.1.0     stringr_1.4.0  
    ## [13] bit64_4.0.5     bit_4.0.4       xfun_0.25       yaml_2.2.1     
    ## [17] fastmap_1.1.0   compiler_4.1.0  pkgconfig_2.0.3 memoise_2.0.0  
    ## [21] htmltools_0.5.2 knitr_1.34
