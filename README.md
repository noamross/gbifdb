
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gbifdb

<!-- badges: start -->

[![R-CMD-check](https://github.com/cboettig/gbifdb/workflows/R-CMD-check/badge.svg)](https://github.com/cboettig/gbifdb/actions)
<!-- badges: end -->

The goal of `gbifdb` is to provide a relational database interface to a
`parquet` based serializations of `gbif` data. Instead of requiring
custom functions for filtering and selecting data from the central GBIF
server (as in `rgbif`), `gbifdb` users can take advantage of the full
array of `dplyr` and `tidyr` functions which can be automatically
translated to SQL by `dbplyr`. Users already familiar with SQL can
construct SQL queries directly with `DBI` instead. `gbifdb` sends these
queries to [`duckdb`](https://duckdb.org), a high-performance,
columnar-oriented database engine which runs entirely inside the client,
(unlike server-client databases such as MySQL or Postgres, no additional
setup is needed outside of installing `gbifdb`.) `duckdb` is able to
execute these SQL queries directly on-disk against the Parquet data
files, side-stepping limitations of available RAM or the need to import
the data. It’s highly optimized implementation can be faster even than
in-memory operations in `dplyr`, especially when using fast SSD-based
storage disks. Unlike the `arrow` R packae interface, which can also
query `parquet` files, `duckdb` supports the full set of SQL
instructions, including windowed operations like `group_by`+`summarise`
as well as table joins.

## Installation

**NOTE**: `gbifdb` currently requires the dev version of `duckdb`, which
you can install using:

``` r
install.packages("https://github.com/duckdb/duckdb/releases/download/master-builds/duckdb_r_src.tar.gz", repos = NULL)
```

<!--

You can install the released version of `gbifdb` from [CRAN](https://CRAN.R-project.org) with:

``` r
install.packages("gbifdb")
```

-->

And the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("cboettig/gbifdb")
```

`gbifdb` has few dependencies: only `duckdb` and `DBI` are required.

## Getting Started

``` r
library(gbifdb)
library(dplyr)  # optional, for dplyr-based operations
```

Before you can use `gbifdb` you will need to download GBIF data.
Alternatively, many users may choose to launch an RStudio-server
instance on a cloud platform that already has a local copy of the GBIF
data available.

Data can be downloaded by registering at GBIF portal, or directly from
various cloud provider copies, including the AWS GBIF public data
catalog, <https://registry.opendata.aws/gbif/>, or the Microsoft Cloud,
<https://planetarycomputer.microsoft.com/dataset/gbif>, which include
directions for direct download or syncing through various client
software.

Once you have downloaded the parquet-formatted GBIF data, simply point
`gbif_conn()` at the directory containing your parquet files to
initialize a connection. (By default, `gbif_conn()` will look for data
in the configurable directory given by `gbif_dir()`).

``` r
conn <- gbif_conn()
```

The resulting connection can be used with `dplyr::tbl()` to access the
full gbif data:

``` r
gbif <- tbl(conn, "gbif")
gbif
#> # Source:   table<gbif> [?? x 50]
#> # Database: duckdb_connection
#>        gbifid datasetkey  occurrenceid   kingdom phylum class order family genus
#>         <dbl> <chr>       <chr>          <chr>   <chr>  <chr> <chr> <chr>  <chr>
#>  1 1572326202 0e2c20a3-3… 7B3E9B63FF90F… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  2 1572326211 0e2c20a3-3… 7B3E9B63FF8BF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  3 1572326213 0e2c20a3-3… 7B3E9B63FF96F… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  4 1572326222 0e2c20a3-3… 7B3E9B63FF8EF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  5 1572326224 0e2c20a3-3… 7B3E9B63FF8FF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  6 1572326210 0e2c20a3-3… 7B3E9B63FF92F… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  7 1572326209 0e2c20a3-3… 7B3E9B63FF8BF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  8 1572326215 0e2c20a3-3… 7B3E9B63FF8FF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#>  9 1572326228 0e2c20a3-3… 7B3E9B63FF8EF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#> 10 1572326205 0e2c20a3-3… 7B3E9B63FF8BF… Animal… Arthr… <NA>  Aran… Capon… Medi…
#> # … with more rows, and 41 more variables: species <chr>,
#> #   infraspecificepithet <chr>, taxonrank <chr>, scientificname <chr>,
#> #   verbatimscientificname <chr>, verbatimscientificnameauthorship <chr>,
#> #   countrycode <chr>, locality <chr>, stateprovince <chr>,
#> #   occurrencestatus <chr>, individualcount <int>, publishingorgkey <chr>,
#> #   decimallatitude <dbl>, decimallongitude <dbl>,
#> #   coordinateuncertaintyinmeters <dbl>, coordinateprecision <dbl>, …
```

``` r
colnames(gbif)
#>  [1] "gbifid"                           "datasetkey"                      
#>  [3] "occurrenceid"                     "kingdom"                         
#>  [5] "phylum"                           "class"                           
#>  [7] "order"                            "family"                          
#>  [9] "genus"                            "species"                         
#> [11] "infraspecificepithet"             "taxonrank"                       
#> [13] "scientificname"                   "verbatimscientificname"          
#> [15] "verbatimscientificnameauthorship" "countrycode"                     
#> [17] "locality"                         "stateprovince"                   
#> [19] "occurrencestatus"                 "individualcount"                 
#> [21] "publishingorgkey"                 "decimallatitude"                 
#> [23] "decimallongitude"                 "coordinateuncertaintyinmeters"   
#> [25] "coordinateprecision"              "elevation"                       
#> [27] "elevationaccuracy"                "depth"                           
#> [29] "depthaccuracy"                    "eventdate"                       
#> [31] "day"                              "month"                           
#> [33] "year"                             "taxonkey"                        
#> [35] "specieskey"                       "basisofrecord"                   
#> [37] "institutioncode"                  "collectioncode"                  
#> [39] "catalognumber"                    "recordnumber"                    
#> [41] "identifiedby"                     "dateidentified"                  
#> [43] "license"                          "rightsholder"                    
#> [45] "recordedby"                       "typestatus"                      
#> [47] "establishmentmeans"               "lastinterpreted"                 
#> [49] "mediatype"                        "issue"
```

Now, we can use `dplyr` to perform standard queries: for instance, the
number of unique species observed by country:

``` r
gbif %>% select(species, countrycode) %>%
  distinct() %>% 
  count(countrycode)
#> # Source:   lazy query [?? x 2]
#> # Database: duckdb_connection
#>    countrycode      n
#>    <chr>        <dbl>
#>  1 US          185986
#>  2 GB           45065
#>  3 HN           12430
#>  4 ES           33794
#>  5 BZ            9912
#>  6 CR           44425
#>  7 AU          122412
#>  8 FI           24912
#>  9 AR           28761
#> 10 CO           42958
#> # … with more rows
```

``` r
growth <- gbif %>% 
  filter(phylum == "Chordata", year > 1990) %>%
  count(class, year) %>% arrange(year)
growth
#> # Source:     lazy query [?? x 3]
#> # Database:   duckdb_connection
#> # Groups:     class
#> # Ordered by: year
#>    class               year      n
#>    <chr>              <int>  <dbl>
#>  1 Cephalaspidomorphi  1991   1152
#>  2 Elasmobranchii      1991  17521
#>  3 Ascidiacea          1991   1602
#>  4 Thaliacea           1991    669
#>  5 Amphibia            1991  18443
#>  6 Sarcopterygii       1991     13
#>  7 <NA>                1991    912
#>  8 Leptocardii         1991     36
#>  9 Actinopterygii      1991 363791
#> 10 Holocephali         1991   1048
#> # … with more rows
```

Recall that when using remote data sources in `dplyr`, the data remains
in the database (i.e. on disk, not in working RAM). This is fine for any
further operations using `dplyr`/`tidyr` functions which can be
translated into SQL. Using such functions we can usually reduce our
resulting table to something much smaller, which can then be pulled into
memory in R for further analysis using `collect()`:

``` r
library(tidyverse)
growth <- collect(growth)

fct_lump_n(growth$class, 6)%>%levels()
#>  [1] "Actinopterygii"     "Amphibia"           "Ascidiacea"        
#>  [4] "Aves"               "Cephalaspidomorphi" "Elasmobranchii"    
#>  [7] "Holocephali"        "Mammalia"           "Myxini"            
#> [10] "Reptilia"           "Thaliacea"          "Other"
top_classes <- growth %>% pull(class) %>% unique() %>% head()

# GBIF: the global bird information facility?
growth %>% 
  mutate(class = fct_lump_n(class, 6)) %>%
  ggplot(aes(year, n, fill=class)) + geom_col() +
  ggtitle("GBIF observations by class")
```

<img src="man/figures/README-unnamed-chunk-8-1.png" width="100%" />
