# A dplyr cube tbl

Produce a [tbl_cube](https://rdrr.io/pkg/cubelyr/man/tbl_cube.html) from
NetCDF. This is a delay-breaking function and causes data to be read
from the source into the tbl cube format defined in the
[dplyr](https://rdrr.io/pkg/cubelyr/man/tbl_cube.html) package.

## Usage

``` r
hyper_tbl_cube(x, ..., force = FALSE)

# S3 method for class 'tidync'
hyper_tbl_cube(x, ..., force = FALSE)

# S3 method for class 'character'
hyper_tbl_cube(x, ..., force = FALSE)
```

## Arguments

- x:

  tidync object

- ...:

  arguments for
  [`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md)

- force:

  ignore caveats about large extraction and just do it

## Value

tbl_cube

`dplyr::tbl_cube`

## Details

The size of an extraction is checked and if *quite large* there is an a
user-controlled prompt to proceed or cancel. This can be disabled with
`options(tidync.large.data.check = FALSE)`

- please see
  [`hyper_array()`](https://docs.ropensci.org/tidync/reference/hyper_array.md)
  for more details.

The tbl cube is a very general and arbitrarily-sized array that can be
used with tidyverse functionality. Dimension coordinates are stored with
the tbl cube, derived from the grid
[transforms](https://docs.ropensci.org/tidync/reference/hyper_transforms.md).

## See also

[`hyper_array()`](https://docs.ropensci.org/tidync/reference/hyper_array.md)
and
[`hyper_tibble()`](https://docs.ropensci.org/tidync/reference/hyper_tibble.md)
which are also delay-breaking functions that cause data to be read

## Examples

``` r
f <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
l3file <- system.file("extdata/oceandata", f, package= "tidync")
(cube <- hyper_tbl_cube(tidync(l3file) |>
activate(chlor_a), lon = lon > 107, lat = abs(lat) < 30))
#> Source: local array [630,720 x 2]
#> D: lon [dbl, 4320]
#> D: lat [dbl, 2160]
#> M: chlor_a [dbl[,720]]
ufile <- system.file("extdata", "unidata", "test_hgroups.nc", 
 package = "tidync", mustWork = TRUE)
 
## some versions of NetCDF don't support this file
## (4.1.3 tidync/issues/82)
group_nc <- try(tidync(ufile), silent = TRUE)
if (!inherits(group_nc, "try-error")) {
 res <-  hyper_tbl_cube(tidync(ufile))
 print(res)
} else {
 ## the error was
 writeLines(c(group_nc))
}
#> Source: local array [74 x 1]
#> D: recNum [int, 74]
#> M: UTC_time [chr[1d]]
```
