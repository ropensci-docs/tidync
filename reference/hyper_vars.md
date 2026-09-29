# Grid status

Functions to report on the current status of the `active` grid.
Information on the active dimensions and variables are listed in a data
frame with multiple columns.

## Usage

``` r
hyper_vars(x, ...)

hyper_dims(x, ...)

hyper_grids(x, ...)
```

## Arguments

- x:

  tidync object

- ...:

  ignored

## Value

data frame

## Details

The dimensions and variables of the active grid are identified in the
[print](https://docs.ropensci.org/tidync/reference/print.tidync.md)
method of the tidync object, these functions exist to provide that
information directly.

`hyper_vars()` will list the ids, data type, name, dimension number,
number of attributes and and coordinate status of the variables on the
currently active grid.

`hyper_dims()` will list the names, lengths, start/count index, ids, and
status of dimensions on the currently active grid. records on the
currently active dimensions.

`hyper_grids()` will list the names, number of dimension, and number of
variables and active status of each grid in the source.

## Examples

``` r
f <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
l3file <- system.file("extdata/oceandata", f, package= "tidync")
tnc <- tidync(l3file)
hyper_vars(tnc)
#> # A tibble: 1 × 6
#>      id name    type     ndims natts dim_coord
#>   <int> <chr>   <chr>    <int> <int> <lgl>    
#> 1     0 chlor_a NC_FLOAT     2    10 FALSE    
hyper_dims(tnc)
#> # A tibble: 2 × 7
#>   name  length start count    id unlim coord_dim
#>   <chr>  <dbl> <int> <int> <int> <lgl> <lgl>    
#> 1 lon     4320     1  4320     1 FALSE TRUE     
#> 2 lat     2160     1  2160     0 FALSE TRUE     
hyper_dims(tnc |> hyper_filter(lat = lat < 20))
#> # A tibble: 2 × 7
#>   name  length start count    id unlim coord_dim
#>   <chr>  <dbl> <int> <int> <int> <lgl> <lgl>    
#> 1 lon     4320     1  4320     1 FALSE TRUE     
#> 2 lat     2160   841  1320     0 FALSE TRUE     
```
