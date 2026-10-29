# Subset NetCDF variable by expression

The `hyper_filter()` acts on a
[tidync](https://docs.ropensci.org/tidync/reference/tidync.md) object by
matching one or more filtering expressions like with
[`dplyr::filter`](https://dplyr.tidyverse.org/reference/filter.html).
This allows us to lazily specify a subset from a NetCDF array without
pulling any data. The modified object may be printed to see the effects
of subsetting, or saved for further use.

## Usage

``` r
hyper_filter(.x, ...)

# S3 method for class 'tidync'
hyper_filter(.x, ...)
```

## Arguments

- .x:

  NetCDF file, connection object, or `tidync` object

- ...:

  currently ignored

## Value

data frame

## Details

The function `hyper_filter()` will act on an existing tidync object or a
source string.

Filter arguments must be named as per the dimensions in the variable in
form `dimname = dimname < 10`. This is a restrictive variant of
[`dplyr::filter()`](https://dplyr.tidyverse.org/reference/filter.html),
with a syntax more like
[`dplyr::mutate()`](https://dplyr.tidyverse.org/reference/mutate.html).
This ensures that each element is named, so we know which dimension to
apply this to, but also that the expression evaluated against can do
some extra work for a nuanced test.

There are special columns provided with each axis, one is 'index' so
that exact matching can be done by position, or to ignore the actual
value of the coordinate. That means we can use a form like
`dimname = index < 10` to subset by position in the array index, without
necessarily knowing the values along that dimension.

## Examples

``` r
f <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
l3file <- system.file("extdata/oceandata", f, package= "tidync")
## filter by value
tidync(l3file) |> hyper_filter(lon = lon < 100)
#> 
#> Data Source (1): S20080012008031.L3m_MO_CHL_chlor_a_9km.nc ...
#> 
#> Grids (4) <dimension family> : <associated variables> 
#> 
#> [1]   D1,D0 : chlor_a    **ACTIVE GRID** ( 9331200  values per variable)
#> [2]   D3,D2 : palette
#> [3]   D0    : lat
#> [4]   D1    : lon
#> 
#> Dimensions 4 (2 active): 
#>   
#>   dim   name  length    min   max start count   dmin  dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>  <dbl> <dbl> <int> <int>  <dbl> <dbl> <lgl> <lgl>     
#> 1 D0    lat     2160  -90.0  90.0     1  2160  -90.0  90.0 FALSE TRUE      
#> 2 D1    lon     4320 -180.  180.      1  3360 -180.  100.0 FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1     3 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1   256 FALSE FALSE     
## filter by index
tidync(l3file) |> hyper_filter(lon = index < 100)
#> 
#> Data Source (1): S20080012008031.L3m_MO_CHL_chlor_a_9km.nc ...
#> 
#> Grids (4) <dimension family> : <associated variables> 
#> 
#> [1]   D1,D0 : chlor_a    **ACTIVE GRID** ( 9331200  values per variable)
#> [2]   D3,D2 : palette
#> [3]   D0    : lat
#> [4]   D1    : lon
#> 
#> Dimensions 4 (2 active): 
#>   
#>   dim   name  length    min   max start count   dmin   dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>  <dbl> <dbl> <int> <int>  <dbl>  <dbl> <lgl> <lgl>     
#> 1 D0    lat     2160  -90.0  90.0     1  2160  -90.0   90.0 FALSE TRUE      
#> 2 D1    lon     4320 -180.  180.      1    99 -180.  -172.  FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1     3 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1   256 FALSE FALSE     

## be careful that multiple comparisons must occur in one expression
 tidync(l3file) |> hyper_filter(lon = lon < 100 & lon > 50)
#> 
#> Data Source (1): S20080012008031.L3m_MO_CHL_chlor_a_9km.nc ...
#> 
#> Grids (4) <dimension family> : <associated variables> 
#> 
#> [1]   D1,D0 : chlor_a    **ACTIVE GRID** ( 9331200  values per variable)
#> [2]   D3,D2 : palette
#> [3]   D0    : lat
#> [4]   D1    : lon
#> 
#> Dimensions 4 (2 active): 
#>   
#>   dim   name  length    min   max start count  dmin  dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>  <dbl> <dbl> <int> <int> <dbl> <dbl> <lgl> <lgl>     
#> 1 D0    lat     2160  -90.0  90.0     1  2160 -90.0  90.0 FALSE TRUE      
#> 2 D1    lon     4320 -180.  180.   2761   600  50.0 100.0 FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1     3 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1   256 FALSE FALSE     

## filter in combination/s
tidync(l3file) |> hyper_filter(lat = abs(lat) < 10, lon = index < 100)
#> 
#> Data Source (1): S20080012008031.L3m_MO_CHL_chlor_a_9km.nc ...
#> 
#> Grids (4) <dimension family> : <associated variables> 
#> 
#> [1]   D1,D0 : chlor_a    **ACTIVE GRID** ( 9331200  values per variable)
#> [2]   D3,D2 : palette
#> [3]   D0    : lat
#> [4]   D1    : lon
#> 
#> Dimensions 4 (2 active): 
#>   
#>   dim   name  length    min   max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>  <dbl> <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    lat     2160  -90.0  90.0   961   240   -9.96    9.96 FALSE TRUE      
#> 2 D1    lon     4320 -180.  180.      1    99 -180.   -172.   FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1     3 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1   256 FALSE FALSE     
```
