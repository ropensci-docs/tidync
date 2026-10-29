# Print tidync data

Print method for the 'tidync_data' list of arrays returned by
[`hyper_array()`](https://docs.ropensci.org/tidync/reference/hyper_array.md).

## Usage

``` r
# S3 method for class 'tidync_data'
print(x, ...)
```

## Arguments

- x:

  'tidync_data' object (from
  [`hyper_array()`](https://docs.ropensci.org/tidync/reference/hyper_array.md))

- ...:

  reserved args

## Value

the input object invisibly

## Details

The output lists the variables and their dimensions of an object from a
previous call to
[`tidync()`](https://docs.ropensci.org/tidync/reference/tidync.md), and
possibly
[`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md).
The available data will differ from the source in terms of variables
(via `select_var` in
[hyper_array](https://docs.ropensci.org/tidync/reference/hyper_array.md))
and the lengths of each dimension (via named expressions in
[`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md)).

## See also

tidync_data

## Examples

``` r
argofile <- system.file("extdata/argo/MD5903593_001.nc", package = "tidync")
argodata <- tidync(argofile) |> hyper_filter(N_LEVELS = index < 5) |> 
              hyper_array(select_var = c("TEMP_ADJUSTED", "PRES"))
print(argodata)
#> Class: tidync_data (list of tidync data arrays)
#> Variables (2): 'TEMP_ADJUSTED', 'PRES'
#> Dimension (2): N_LEVELS,N_PROF (4, 2)
#> Source: /github/home/R/x86_64-pc-linux-gnu-library/4.6/tidync/extdata/argo/MD5903593_001.nc
```
