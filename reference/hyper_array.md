# Extract NetCDF data as an array

Extract the raw array data as a list of one or more arrays. This can be
the entire variable/s or after dimension-slicing using
[`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md)
expressions. This is a delay-breaking function and causes data to be
read from the source into R native arrays. This list of arrays is
lightly classed as tidync_data, with methods for
[`print()`](https://rdrr.io/r/base/print.html) and
[`tidync()`](https://docs.ropensci.org/tidync/reference/tidync.md).

## Usage

``` r
hyper_array(
  x,
  select_var = NULL,
  ...,
  raw_datavals = FALSE,
  force = FALSE,
  drop = TRUE
)

hyper_slice(
  x,
  select_var = NULL,
  ...,
  raw_datavals = FALSE,
  force = FALSE,
  drop = TRUE
)

# S3 method for class 'tidync'
hyper_array(
  x,
  select_var = NULL,
  ...,
  raw_datavals = FALSE,
  force = FALSE,
  drop = TRUE
)

# S3 method for class 'character'
hyper_array(
  x,
  select_var = NULL,
  ...,
  raw_datavals = FALSE,
  force = FALSE,
  drop = TRUE
)
```

## Arguments

- x:

  NetCDF file, connection object, or
  [tidync](https://docs.ropensci.org/tidync/reference/tidync.md) object

- select_var:

  optional vector of variable names to select

- ...:

  passed to
  [`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md)

- raw_datavals:

  logical to control whether scaling in the NetCDF is applied or not

- force:

  ignore caveats about large extraction and just do it

- drop:

  collapse degenerate dimensions, defaults to `TRUE`

## Value

a named list of arrays classed as `tidync_data`, one element per
variable, with dimension coordinates stored in the `transforms`
attribute

## Details

The function `hyper_array()` is used by
[`hyper_tibble()`](https://docs.ropensci.org/tidync/reference/hyper_tibble.md)
and
[`hyper_tbl_cube()`](https://docs.ropensci.org/tidync/reference/hyper_tbl_cube.md)
to actually extract data arrays from NetCDF, if a result would be
particularly large there is a check made and user-opportunity to cancel.
This is controllable as an option
`getOption('tidync.large.data.check')`, and can be set to never check
with `options(tidync.large.data.check = FALSE)`.

The function `hyper_array()` will act on an existing tidync object or a
source string.

By default all variables in the active grid are returned, use
`select_var` to specify one or more desired variables.

The transforms are stored as a list of tables in an attribute
`transforms`, access these with
[`hyper_transforms()`](https://docs.ropensci.org/tidync/reference/hyper_transforms.md).

## See also

[print.tidync_data](https://docs.ropensci.org/tidync/reference/print.tidync_data.md)
for a description of the print summary,
[`hyper_tbl_cube()`](https://docs.ropensci.org/tidync/reference/hyper_tbl_cube.md)
and
[`hyper_tibble()`](https://docs.ropensci.org/tidync/reference/hyper_tibble.md)
which are also delay-breaking functions that cause data to be read

## Examples

``` r
f <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
l3file <- system.file("extdata/oceandata", f, package= "tidync")

## extract a raw list by filtered dimension
library(dplyr)
#> 
#> Attaching package: ‘dplyr’
#> The following objects are masked from ‘package:stats’:
#> 
#>     filter, lag
#> The following objects are masked from ‘package:base’:
#> 
#>     intersect, setdiff, setequal, union
araw1 <- tidync(l3file) |>
 hyper_filter(lat = between(lat, -78, -75.8), 
              lon = between(lon, 165, 171)) |>
 hyper_array()

araw <- tidync(l3file) |> 
         hyper_filter(lat = abs(lat) < 10, 
                     lon = index < 100) |>
  hyper_array()

## hyper_array will pass the expressions to hyper_filter
braw <- tidync(l3file) |> 
  hyper_array(lat = abs(lat) < 10, lon = index < 100)

## get the transforms tables (the axis coordinates)
lapply(attr(braw, "transforms"), 
   function(x) nrow(dplyr::filter(x, selected)))
#> $lon
#> [1] 99
#> 
#> $lat
#> [1] 240
#> 
## the selected axis coordinates should match in order and in size
lapply(braw, dim)
#> $chlor_a
#> [1]  99 240
#> 
```
