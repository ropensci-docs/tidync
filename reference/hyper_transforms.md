# Axis transforms

Axis 'transforms' are data frames of each dimension in a NetCDF source.
`hyper_transforms` returns a list of the active transforms by default,

## Usage

``` r
hyper_transforms(x, all = FALSE, ...)

# Default S3 method
hyper_transforms(x, all = FALSE, ...)
```

## Arguments

- x:

  tidync object

- all:

  set to `TRUE` to return all transforms, not only active ones

- ...:

  ignored

## Value

list of axis transforms

## Details

Each transform is available by name from a list, and each data frame has
the coordinate of the dimension, its index, and a 'selected' variable
set by the filtering expressions in `hyper_filter` and used by the
read-functions `hyper_array` and `hyper_tibble`.

Use `hyper_transforms` to interrogate and explore the available
dimension manually, or for development of custom functions.

## Examples

``` r
l3file <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
f <- system.file("extdata", "oceandata", l3file, package = "tidync")
ax <- tidync(f) |> hyper_transforms()
names(ax)
#> [1] "lon" "lat"
lapply(ax, dim)
#> $lon
#> [1] 4320    6
#> 
#> $lat
#> [1] 2160    6
#> 

## this function returns the transforms tidync knows about for this source
str(tidync(f)$transforms)
#> List of 4
#>  $ lon          : tibble [4,320 × 6] (S3: tbl_df/tbl/data.frame)
#>   ..$ lon      : num [1:4320] -180 -180 -180 -180 -180 ...
#>   ..$ index    : int [1:4320] 1 2 3 4 5 6 7 8 9 10 ...
#>   ..$ id       : int [1:4320] 1 1 1 1 1 1 1 1 1 1 ...
#>   ..$ name     : chr [1:4320] "lon" "lon" "lon" "lon" ...
#>   ..$ coord_dim: logi [1:4320] TRUE TRUE TRUE TRUE TRUE TRUE ...
#>   ..$ selected : logi [1:4320] TRUE TRUE TRUE TRUE TRUE TRUE ...
#>  $ lat          : tibble [2,160 × 6] (S3: tbl_df/tbl/data.frame)
#>   ..$ lat      : num [1:2160] 90 89.9 89.8 89.7 89.6 ...
#>   ..$ index    : int [1:2160] 1 2 3 4 5 6 7 8 9 10 ...
#>   ..$ id       : int [1:2160] 0 0 0 0 0 0 0 0 0 0 ...
#>   ..$ name     : chr [1:2160] "lat" "lat" "lat" "lat" ...
#>   ..$ coord_dim: logi [1:2160] TRUE TRUE TRUE TRUE TRUE TRUE ...
#>   ..$ selected : logi [1:2160] TRUE TRUE TRUE TRUE TRUE TRUE ...
#>  $ eightbitcolor: tibble [256 × 6] (S3: tbl_df/tbl/data.frame)
#>   ..$ eightbitcolor: int [1:256] 1 2 3 4 5 6 7 8 9 10 ...
#>   ..$ index        : int [1:256] 1 2 3 4 5 6 7 8 9 10 ...
#>   ..$ id           : int [1:256] 3 3 3 3 3 3 3 3 3 3 ...
#>   ..$ name         : chr [1:256] "eightbitcolor" "eightbitcolor" "eightbitcolor" "eightbitcolor" ...
#>   ..$ coord_dim    : logi [1:256] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ selected     : logi [1:256] TRUE TRUE TRUE TRUE TRUE TRUE ...
#>  $ rgb          : tibble [3 × 6] (S3: tbl_df/tbl/data.frame)
#>   ..$ rgb      : int [1:3] 1 2 3
#>   ..$ index    : int [1:3] 1 2 3
#>   ..$ id       : int [1:3] 2 2 2
#>   ..$ name     : chr [1:3] "rgb" "rgb" "rgb"
#>   ..$ coord_dim: logi [1:3] FALSE FALSE FALSE
#>   ..$ selected : logi [1:3] TRUE TRUE TRUE
names(hyper_transforms(tidync(f), all = TRUE))
#> [1] "lon"           "lat"           "eightbitcolor" "rgb"          
```
