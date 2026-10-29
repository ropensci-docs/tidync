# Activate a NetCDF grid

A grid in NetCDF is a particular shape and size available for array
variables, and consists of sets of dimensions. To activate a grid is to
set the context for downstream operations, for querying, summarizing and
reading data. There's no sense in performing these operations on more
than one grid at a time, but multiple variables may exist in a single
grid. There may be only one significant grid in a source or many,
individual dimensions are themselves grids.

## Usage

``` r
activate(.data, what, ..., select_var = NULL)

# S3 method for class 'tidync'
activate(.data, what, ..., select_var = NULL)

# S3 method for class 'tidync'
active(x)

active(x)

# Default S3 method
active(x)

active(x) <- value

# Default S3 method
active(x) <- value
```

## Arguments

- .data:

  NetCDF object

- what:

  name of a grid or variable

- ...:

  reserved, currently ignored

- select_var:

  optional argument to set selected state of variable/s by name

- x:

  NetCDF object

- value:

  name of grid or variable to be active

## Value

NetCDF object

## Details

There may be more than one grid and one is always activated by default.
A grid may be activated by name in the form of 'D1,D0' where one or more
numbered dimensions indicates the grid. The grid definition names are
printed as part of the summary of in the tidync object and may be
obtained directly with
[`hyper_grids()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
on the tidync object.

Activation of a grid sets the context for downstream operations (slicing
and reading data) from NetCDF, and as there may be several grids in a
single source activation allows a different choice of available
variables. By default the largest grid is activated. Once activated, all
downstream tasks apply to the set of variables that exist on that grid.

If `activate()` is called with a variable name, it puts the variable
first. The function `active()` gets and sets the active grid. To
restrict ultimate read to particular variables use the `select_var`
argument to
[`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md),
[`hyper_tibble()`](https://docs.ropensci.org/tidync/reference/hyper_tibble.md)
and
[`hyper_tbl_cube()`](https://docs.ropensci.org/tidync/reference/hyper_tbl_cube.md).

Scalar variables are not currently available to tidync, and it's not
obvious how activation would occur for scalars, but in future perhaps
`activate("S")` could be the right way forward.

## See also

hyper_filter hyper_tibble hyper_tbl_cube

## Examples

``` r
if (!tolower(Sys.info()[["sysname"]]) == "sunos") {
 l3file <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
 rnc <- tidync(system.file("extdata", "oceandata", l3file,
 package = "tidync"))
 activate(rnc, "palette")

 ## extract available grid names
 hyper_grids(rnc)
}
#> # A tibble: 4 × 4
#>   grid  ndims nvars active
#>   <chr> <int> <int> <lgl> 
#> 1 D1,D0     2     1 TRUE  
#> 2 D3,D2     2     1 FALSE 
#> 3 D0        1     1 FALSE 
#> 4 D1        1     1 FALSE 
```
