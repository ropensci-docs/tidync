# Tidy NetCDF

Connect to a NetCDF source and allow use of `hyper_*` verbs for slicing
with
[`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md),
extracting data with
[`hyper_array()`](https://docs.ropensci.org/tidync/reference/hyper_array.md)
and \[hyper_tibble() from an activated grid. By default the largest
*grid* encountered is activated,
see[`activate()`](https://docs.ropensci.org/tidync/reference/activate.md).

## Usage

``` r
tidync(x, what, ...)

# S3 method for class 'character'
tidync(x, what, ..., concat_dim = NULL, fast = FALSE)

# S3 method for class 'tidync_data'
tidync(x, what, ...)
```

## Arguments

- x:

  path to a NetCDF file, or a character vector of paths for multi-source
  access (requires `concat_dim`)

- what:

  (optional) character name of grid (see
  [`ncmeta::nc_grids`](https://hypertidy.github.io/ncmeta/reference/nc_grids.html))
  or (bare) name of variable (see
  [`ncmeta::nc_vars`](https://hypertidy.github.io/ncmeta/reference/nc_vars.html))
  or index of grid to `activate`

- ...:

  reserved for arguments to methods, currently ignored

- concat_dim:

  (optional) name of the dimension to concatenate across sources, or a
  list with elements `name` (dimension name) and `values` (vector of
  coordinate values, one per source). The list form avoids opening files
  2..N entirely — useful when values are already known from a file
  database. Values can be numeric, Date, or POSIXct.

- fast:

  logical, if `TRUE` skip full metadata validation for sources after the
  first (only reads the `concat_dim` coordinate). Mismatches are
  detected at data-read time. Ignored when `concat_dim$values` is
  supplied.

## Value

a `tidync` object

## Details

The print method for tidync includes a lot of information about which
variables exist on which dimensions, and if any slicing
([`hyper_filter()`](https://docs.ropensci.org/tidync/reference/hyper_filter.md))
operations have occurred these are summarized as 'start' and 'count'
modifications relative to the dimension lengths. See
[print](https://docs.ropensci.org/tidync/reference/print.tidync.md) for
these details, and
[hyper_vars](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
for programmatic access to this information

Many NetCDF forms are supported and tidync tries to reduce the
interpretation applied to a given source. The NetCDF system defines a
'grid' for storing array data, where 'grid' is the array 'shape', or
'set of dimensions'). There may be several grids in a single source and
so we introduce the concept of grid 'activation'. Once activated, all
downstream tasks apply to the set of variables that exist on that grid.

NetCDF sources with numeric types are chosen by default, even if
existing 'NC_CHAR' type variables are on the largest grid. When read any
'NC_CHAR' type variables are exploded into single character elements so
that dimensions match the source.

## Multi-source

When `x` is a character vector of length \> 1 and `concat_dim` is
specified, tidync builds a consolidated view across all sources. The
first source is used as a template for grid structure, and the
`concat_dim` coordinate values are read from each source and
concatenated. Downstream operations (`hyper_filter`, `hyper_array`,
etc.) work transparently across the collection.

Use `fast = TRUE` for large collections to skip full metadata validation
of sources 2..N (only the `concat_dim` coordinate is read). Mismatches
are detected lazily at data-read time.

For maximum speed, supply coordinate values directly via the list form
`concat_dim = list(name = "time", values = <vector>)`. This opens only
the first source (for the template) and builds the concat transform from
the supplied values with zero additional file I/O. This is ideal when
coordinate values are already available from a file database.

## Grids

A grid is an instance of a particular set of dimensions, which can be
shared by more than one variable. This is not the 'rank' of a variable
(the number of dimensions) since a single data set may have many 3D
variables composed of different sets of axes/dimensions. There's no
formality around the concept of 'shape', as far as we know.

A dimension may have length zero, but this is a special case for a
"measure" dimension, we think. (It doesn't mean the product of the
dimensions is zero, for example).

## Limitations

Files with compound types are not yet supported and should fail
gracefully. Groups are not yet supported.

We haven't yet explored 'HDF5' in detail, so any feedback is
appreciated. Major use of compound types is made by
<https://github.com/sosoc/croc>.

## Examples

``` r
## a SeaWiFS (S) Level-3 Mapped (L3m) monthly (MO) chlorophyll-a (CHL)
## remote sensing product at 9km resolution (at the equator)
## from the NASA ocean colour group in NetCDF4 format (.nc)
## for 31 day period January 2008 (S20080012008031) 
f <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
l3file <- system.file("extdata/oceandata", f, package= "tidync")
## skip on Solaris
if (!tolower(Sys.info()[["sysname"]]) == "sunos") {
tnc <- tidync(l3file)
print(tnc)
}
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
#> 2 D1    lon     4320 -180.  180.      1  4320 -180.  180.  FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name          length   min   max unlim coord_dim 
#>   <chr> <chr>          <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D2    rgb                3     1     3 FALSE FALSE     
#> 2 D3    eightbitcolor    256     1   256 FALSE FALSE     

## very simple Unidata example file, with one dimension
if (FALSE) { # \dontrun{
uf <- system.file("extdata/unidata", "test_hgroups.nc", package = "tidync")
recNum <- tidync(uf) |> hyper_tibble()
print(recNum)
} # }
## a raw grid of Southern Ocean sea ice concentration from IFREMER
## it is 12.5km resolution passive microwave concentration values
## on a polar stereographic grid, on 2 October 2017, displaying the 
## "hole in the ice"
ifr <- system.file("extdata/ifremer", "20171002.nc", package = "tidync")
ifrnc <- tidync(ifr)
ifrnc |> hyper_tibble(select_var = "concentration")
#> # A tibble: 207,195 × 4
#>    concentration    ni    nj time               
#>            <int> <int> <int> <chr>              
#>  1             0   291     5 2017-10-02T12:00:00
#>  2             0   292     5 2017-10-02T12:00:00
#>  3             0   293     5 2017-10-02T12:00:00
#>  4             0   294     5 2017-10-02T12:00:00
#>  5             0   295     5 2017-10-02T12:00:00
#>  6             0   296     5 2017-10-02T12:00:00
#>  7             0   297     5 2017-10-02T12:00:00
#>  8             0   298     5 2017-10-02T12:00:00
#>  9             0   299     5 2017-10-02T12:00:00
#> 10             0   300     5 2017-10-02T12:00:00
#> # ℹ 207,185 more rows

## multi-source: concatenate files along a dimension
if (FALSE) { # \dontrun{
files <- c("sst_2020_01.nc", "sst_2020_02.nc", "sst_2020_03.nc")
tnc <- tidync(files, concat_dim = "time")
tnc
## filter and read across all sources transparently
tnc |> hyper_filter(time = time > 18300) |> hyper_tibble()

## fast mode for large collections (skips full metadata scan)
all_files <- list.files("daily/", pattern = "\\.nc$", full.names = TRUE)
tnc_fast <- tidync(all_files, concat_dim = "time", fast = TRUE)

## zero file I/O: supply values from a file database (e.g. raadfiles)
## only the first file is opened (for the template)
dates <- as.Date(c("2020-01-01", "2020-01-02", "2020-01-03"))
tnc_db <- tidync(files, concat_dim = list(name = "time", values = dates))
## filter directly on the values you supplied
tnc_db |> hyper_filter(time = time > as.Date("2020-01-01")) |> hyper_tibble()
} # }
```
