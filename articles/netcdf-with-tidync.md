# NetCDF with tidync

The goal of tidync is to ease exploring the contents of a NetCDF source
and to simplify the process of data extraction.

Array dimension expressions virtually *slice* (i.e. *index*) into data
arrays with immediate feedback, and ease programming for automating data
extraction. No data is read until explicitly requested, and can be
pulled directly as an array, or in long-form as a data frame.

NetCDF is **Network Common Data Form**, a [data
model](https://twitter.com/TedHabermann/status/958034585002041344) and
an
[API](https://en.wikipedia.org/wiki/Application_programming_interface).

This is a very common, and *very general* way to store and work with
scientific array-based data. NetCDF is defined and provided by
[Unidata](https://www.unidata.ucar.edu/software/netcdf).

Here we introduce traditional concepts of NetCDF, and show examples with
built-in files and online sources to demonstrate tidync functionality.

## NetCDF

NetCDF is a very widely used file format for storing array-based data as
*variables*. The **space** occupied by a **variable** is defined by its
**dimensions** and their metadata. Dimensions are by definition
*one-dimensional* (i.e. an atomic vector in R of length 1 or more), an
array with coordinate metadata, units, type and interpretation. The
**space** of a variable is defined as one or more of the dimensions in
the file. A given variable won’t necessarily use all the available
dimensions and no dimensions are mandatory or particularly special.

Some conventions exist to define usage and minimal standards for
metadata for particular file schemas, but these are many and varied, and
not always adhered to. Tidync does not provide a data interpretation as
it is intended for general use, including by tools that create data
formats.

The R community is not particularly strong with use of NetCDF, though it
is common and widely used it pales compared to use in general climate
science work, and there the most used tool is the [CDO Climate Data
Operators](https://code.mpimet.mpg.de/projects/cdo). In R the most
common tools used are ncdf4 and raster (which uses ncdf4).

Both the RNetCDF and ncdf4 packages provide a traditional summary
format, familiar to many NetCDF users as the output of the command line
program
[`ncdump`](https://docs.unidata.ucar.edu/nug/current/netcdf_utilities_guide.html).

``` r

ice_file <- system.file("extdata", "ifremer", "20171002.nc", package = "tidync", mustWork = TRUE)
library(RNetCDF)
print.nc(open.nc(ice_file))
#> netcdf classic {
#> dimensions:
#>  ni = 632 ;
#>  nj = 664 ;
#>  time = 1 ;
#> variables:
#>  NC_INT time(time) ;
#>      NC_CHAR time:long_name = "time" ;
#>      NC_CHAR time:units = "hours since 1900-1-1 0:0:0" ;
#>  NC_BYTE concentration(ni, nj, time) ;
#>      NC_CHAR concentration:long_name = "sea-ice concentration" ;
#>      NC_CHAR concentration:units = "percent" ;
#>      NC_DOUBLE concentration:scale_factor = 1 ;
#>      NC_DOUBLE concentration:add_offset = 0 ;
#>      NC_BYTE concentration:missing_value = -128 ;
#>      NC_BYTE concentration:_FillValue = -128 ;
#>  NC_BYTE quality_flag(ni, nj, time) ;
#>      NC_CHAR quality_flag:long_name = "quality_flag" ;
#>      NC_CHAR quality_flag:units = "n/a" ;
#>      NC_DOUBLE quality_flag:scale_factor = 1 ;
#>      NC_DOUBLE quality_flag:add_offset = 0 ;
#>      NC_BYTE quality_flag:missing_value = -128 ;
#>      NC_BYTE quality_flag:_FillValue = -128 ;
#> 
#> // global attributes:
#>      NC_CHAR :CONVENTIONS = "COARDS" ;
#>      NC_CHAR :long_name = "Sea-ice concentration as observed by SSM/I" ;
#>      NC_CHAR :short_name = "PSI-F18-Concentration" ;
#>      NC_CHAR :producer_agency = "IFREMER" ;
#>      NC_CHAR :producer_institution = "CERSAT" ;
#>      NC_CHAR :netcdf_version_id = "3.4" ;
#>      NC_CHAR :product_version = "2.0" ;
#>      NC_CHAR :creation_time = "2017-280T08:26:34.000" ;
#>      NC_CHAR :time_resolution = "daily" ;
#>      NC_CHAR :grid = "NSIDC" ;
#>      NC_CHAR :pole = "south" ;
#>      NC_CHAR :spatial_resolution = "12.5 km" ;
#>      NC_CHAR :platform_id = "F18" ;
#>      NC_CHAR :instrument = "SSM/I" ;
#> }
```

Using `ncdump` at the command line on a suitable system would yield very
similar output to the print above.

``` bash
ncdump -h /path/to/extdata/ifremer/20171002.nc
```

With the ncdf4 package it’s a slightly different approach, but gives the
same result.

``` r

print(ncdf4::nc_open(ice_file))
```

Notice how the listing above is organized by *dimension* and then by
*variable*. It’s not particularly obvious that some variables are
defined within the same set of dimensions as others.

A NetCDF file is a container for simple array based data structures.
There is limited capacity [^1] in the formal API for accessing data
randomly within a variable, the primary mechanism is to define offset
and stride (start and count) hyperslab indexes.

### tidync

Tidync aims to ease exploration of the contents of a NetCDF file and
provides methods extract arbitrary hyperslabs. These can be used
directly in array contexts, or in “long form” database contexts.

On first contact with the file, the available variables are classified
by grid and dimension. The “active” grid is the one that queries may be
made against, and may be changed with the `activate` function.

``` r

library(tidync)
tidync(ice_file)
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1   632       1     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1   664       1     664 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

Here we see variables are clearly grouped by the *grid* they exist in,
where grid is a specific (and ordered!) set of dimensions. This allows
us to see the set of variables that implicitly co-exist, they have the
same *shape*. The first grid “D0,D1,D2” has two variables,
*concentration* and *quality_flag*, and the second “D2” has only one
variable *time*. There are no general rules here, a file might have any
number of dimensions and variables, and any variable might be defined by
one or more dimensions.

In this case the D2 grid has only one variable in its single dimension,
and it happens to be a special kind of variable - a “coordinate
dimension”, as indicated by the `coord_dim` flag. In the traditional
`ncdump` summary above it’s easy to see there’s only really one data
grid, in `ni,nj,time` that it holds two variables, and that time is a
special coordinate dimension - in contrast neither `ni` or `nj` have an
explicit 1-dimension variable. When there are many dimensions and/or
many variables those patterns are *not* easy to see.

A particular grid was chosen by default, this is the “D0,D1,D2” grid
composed of 3 dimensions, generally the largest grid will be chosen as
that is *usually* the target we would be after. To choose a different
grid we may nominate it by name, or by member variable.

By name we choose a grid composed of only one dimension, and the summary
print makes a distinction based on which dimensions are *active*.

``` r

tidync(ice_file) |> activate("D2")
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag
#> [2]   D2       : time    **ACTIVE GRID** ( 1  value per variable)
#> 
#> Dimensions 3 (1 active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name  length   min   max unlim coord_dim 
#>   <chr> <chr>  <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D0    ni       632     1   632 FALSE FALSE     
#> 2 D1    nj       664     1   664 FALSE FALSE
```

It is also possible to choose a grid by *variable name*. If we choose a
variable in the default grid, then it’s no different to the first case.

``` r

tidync(ice_file) |> activate("time")
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag
#> [2]   D2       : time    **ACTIVE GRID** ( 1  value per variable)
#> 
#> Dimensions 3 (1 active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE      
#>   
#> Inactive dimensions:
#>   
#>   dim   name  length   min   max unlim coord_dim 
#>   <chr> <chr>  <dbl> <dbl> <dbl> <lgl> <lgl>     
#> 1 D0    ni       632     1   632 FALSE FALSE     
#> 2 D1    nj       664     1   664 FALSE FALSE

## choose grid by variable name, which happens to be the default grid here
tidync(ice_file) |> activate("quality_flag")
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1   632       1     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1   664       1     664 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE

## same as the default
tidync(ice_file) |> activate("D0,D1,D2")
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1   632       1     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1   664       1     664 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

The data variables available on a grid can be expanded out as a single
data frame, which all the coordinates copied out - this is not
efficient(!) but if we craft our queries sensibly to read only what we
need, it’s a very easy way to explore the data in a file.

The ‘hyper_filter’ function allows specification of expressions to
subset a variable based on each dimension’s coordinate values.

If no expressions are included we are presented with a table containing
a row for each dimension, its extent in coordinates and its length. For
convenience we also assign the activate form to an R variable, though we
could just chain the entire operation without this.

``` r

concentration <- tidync(ice_file) 
concentration |> hyper_filter() 
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1   632       1     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1   664       1     664 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

By specifying inequality expressions we see an *implicit* subsetting of
the array. Everything so far is implicit to delay any file-based
computation required to actually interact with the file and read from
it.

Notice that these are “name = expr” paired expressions, because the
right hand side may be quite general we need the left hand side name to
be assured of the name of the dimension referred to.

``` r

concentration |> hyper_filter(nj = nj < 20)
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1   632       1     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1    19       1      19 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

We can also use the special internal variable ‘index’, which will test
against position in the dimension elements ‘1:length’ rather than the
values. It’s not different in this case because ni and nj are just
position dimensions anyway. The special ‘dplyr’ adverbs like ‘between’
will work.

``` r

concentration |> 
  hyper_filter(ni = index < 50, 
               nj = dplyr::between(index, 30, 100))
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632     1    49       1      49 FALSE FALSE     
#> 2 D1    nj       664       1     664    30    71      30     100 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

### Data extraction

How to use these idioms to extract actual data?

We can now exercise these variable choice and dimension filters to
return actual data, either in by slicing out a “slab” in array-form, or
as a data frame.

``` r

hf <- concentration |> 
  hyper_filter(ni = index > 150, 
               nj = dplyr::between(index, 30, 100))

## as an array
arr <- hf |> hyper_array()
str(arr)
#> List of 2
#>  $ concentration: int [1:482, 1:71] NA NA NA NA NA NA NA NA NA NA ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ ni: chr [1:482] "151" "152" "153" "154" ...
#>   .. ..$ nj: chr [1:71] "30" "31" "32" "33" ...
#>  $ quality_flag : int [1:482, 1:71] 8 8 8 8 8 8 8 8 8 8 ...
#>   ..- attr(*, "dimnames")=List of 2
#>   .. ..$ ni: chr [1:482] "151" "152" "153" "154" ...
#>   .. ..$ nj: chr [1:71] "30" "31" "32" "33" ...
#>  - attr(*, "transforms")=List of 3
#>   ..$ ni  : tibble [632 × 6] (S3: tbl_df/tbl/data.frame)
#>   .. ..$ ni       : int [1:632] 1 2 3 4 5 6 7 8 9 10 ...
#>   .. ..$ index    : int [1:632] 1 2 3 4 5 6 7 8 9 10 ...
#>   .. ..$ id       : int [1:632] 0 0 0 0 0 0 0 0 0 0 ...
#>   .. ..$ name     : chr [1:632] "ni" "ni" "ni" "ni" ...
#>   .. ..$ coord_dim: logi [1:632] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   .. ..$ selected : logi [1:632] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ nj  : tibble [664 × 6] (S3: tbl_df/tbl/data.frame)
#>   .. ..$ nj       : int [1:664] 1 2 3 4 5 6 7 8 9 10 ...
#>   .. ..$ index    : int [1:664] 1 2 3 4 5 6 7 8 9 10 ...
#>   .. ..$ id       : int [1:664] 1 1 1 1 1 1 1 1 1 1 ...
#>   .. ..$ name     : chr [1:664] "nj" "nj" "nj" "nj" ...
#>   .. ..$ coord_dim: logi [1:664] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   .. ..$ selected : logi [1:664] FALSE FALSE FALSE FALSE FALSE FALSE ...
#>   ..$ time: tibble [1 × 7] (S3: tbl_df/tbl/data.frame)
#>   .. ..$ time     : num 1032204
#>   .. ..$ timestamp: chr "2017-10-02T12:00:00"
#>   .. ..$ index    : int 1
#>   .. ..$ id       : int 2
#>   .. ..$ name     : chr "time"
#>   .. ..$ coord_dim: logi TRUE
#>   .. ..$ selected : logi TRUE
#>  - attr(*, "source")= tibble [1 × 2] (S3: tbl_df/tbl/data.frame)
#>   ..$ access: POSIXct[1:1], format: "2026-08-30 09:46:57"
#>   ..$ source: chr "/github/home/R/x86_64-pc-linux-gnu-library/4.6/tidync/extdata/ifremer/20171002.nc"
#>  - attr(*, "class")= chr "tidync_data"

## as a data frame
hf |> 
  hyper_tibble() |> 
  dplyr::filter(!is.na(concentration)) |> dplyr::distinct(concentration, quality_flag)
#> # A tibble: 105 × 2
#>    concentration quality_flag
#>            <int>        <int>
#>  1             0            0
#>  2             0           12
#>  3            42            0
#>  4            20            0
#>  5            21            0
#>  6            55            0
#>  7            17            0
#>  8            64            0
#>  9            25            0
#> 10            23            0
#> # ℹ 95 more rows
```

### Interrogating data by dimensions

The connection object ‘hf’ is available for determining what is
available and how we might cut into it. ‘time’ interestingly is of
length 1, so perhaps adds nothing to the information about this
otherwise 2D data set. If we were to open this file in `ncdf4` or
`RNetCDF` and wanted to take a subset of the file out, we would have to
specify and `start` of 1 and a `count` of \` even though it’s completely
redundant.

``` r

hf
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632   151   482     151     632 FALSE FALSE     
#> 2 D1    nj       664       1     664    30    71      30     100 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

The ‘start’ and ‘count’ values reported are directly useable by the
traditional API tools, and in particular by the functions
[`ncdf4::ncvar_get()`](https://rdrr.io/pkg/ncdf4/man/ncvar_get.html)
(`varid`, `start`, `count`), its counterpart in
[`RNetCDF::var.get.nc()`](https://rdrr.io/pkg/RNetCDF/man/var.get.nc.html)
and command line tools like CDO.

But, it’s a pain to have to know the dimension of the variable and
specify every slot. Code in the traditional API that looks like this

``` r

## WARNING, pseudocode
var_get(con, variable, start = c(1, 1, 1), count = c(10, 5, 1))
```

Obviously, it should be reasonable to specify only the count of any
dimensions that *we don’t want the entirety of*. This problem manifests
exactly in R arrays generally, we can’t provide information only about
the dimensions we want, they have to be specified explicitly - even if
we mean *all of it*.

It does not matter what we include in the filter query, it can be all,
some or none of the available dimensions, in any order.

``` r

hf |> hyper_filter(nj = index < 20, ni = ni > 20)
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632    21   612      21     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1    19       1      19 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE

hf |> hyper_filter(nj = index < 20)
#> 
#> Data Source (1): 20171002.nc ...
#> 
#> Grids (2) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D1,D2 : concentration, quality_flag    **ACTIVE GRID** ( 419648  values per variable)
#> [2]   D2       : time
#> 
#> Dimensions 3 (all active): 
#>   
#>   dim   name  length     min     max start count    dmin    dmax unlim coord_dim 
#>   <chr> <chr>  <dbl>   <dbl>   <dbl> <int> <int>   <dbl>   <dbl> <lgl> <lgl>     
#> 1 D0    ni       632       1     632   151   482     151     632 FALSE FALSE     
#> 2 D1    nj       664       1     664     1    19       1      19 FALSE FALSE     
#> 3 D2    time       1 1032204 1032204     1     1 1032204 1032204 FALSE TRUE
```

The other requirement we have is the ability to automate these task, and
so far we have only interacted with the dimensionality information from
the print out. For programming, there are functions
[`hyper_vars()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md),
[`hyper_dims()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
and
[`hyper_grids()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
to report on elements in the source. The value of
[`hyper_vars()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
and
[`hyper_dims()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
is restricted to the *active* grid. The function
[`hyper_grids()`](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
reports all available grids, with the currently active one indicated.
The name of the current grid is also available via
[`active()`](https://docs.ropensci.org/tidync/reference/activate.md).

``` r

hyper_vars(hf)
#> # A tibble: 2 × 6
#>      id name          type    ndims natts dim_coord
#>   <int> <chr>         <chr>   <int> <int> <lgl>    
#> 1     1 concentration NC_BYTE     3     6 FALSE    
#> 2     2 quality_flag  NC_BYTE     3     6 FALSE
hyper_dims(hf)
#> # A tibble: 3 × 7
#>   name  length start count    id unlim coord_dim
#>   <chr>  <dbl> <int> <int> <int> <lgl> <lgl>    
#> 1 ni       632   151   482     0 FALSE FALSE    
#> 2 nj       664    30    71     1 FALSE FALSE    
#> 3 time       1     1     1     2 FALSE TRUE

## change the active grid
hf |> activate("D2") |> 
  hyper_vars()
#> # A tibble: 1 × 6
#>      id name  type   ndims natts dim_coord
#>   <int> <chr> <chr>  <int> <int> <lgl>    
#> 1     0 time  NC_INT     1     2 TRUE

active(hf)
#> [1] "D0,D1,D2"

hf |> activate("D2") |>
  active()
#> [1] "D2"
```

### Transforms

Under the hood tidync manages the relationship between dimensions and
coordinates via *transforms* tables. This is a 1-dimensional mapping
between dimension index and its coordinate value (if there’s no explicit
value it is assumed to be equal to the 1-based index). These tables are
available via the
[`hyper_transforms()`](https://docs.ropensci.org/tidync/reference/hyper_transforms.md)
function.

Each table has columns `<dimension-name>`, `index`, `id`, `name`,
`coord_dim` (whether the dimension has explicit coordinates, in the
`<dimension-name>` column), and `selected`. The `selected` column
records which of the dimension elements is currently requested by a
`hyper_filter` query, and by default is set to `TRUE`. Expressions in
the filter function work by updating this column.

``` r

hyper_transforms(hf)
#> $ni
#> # A tibble: 632 × 6
#>       ni index    id name  coord_dim selected
#>    <int> <int> <int> <chr> <lgl>     <lgl>   
#>  1     1     1     0 ni    FALSE     FALSE   
#>  2     2     2     0 ni    FALSE     FALSE   
#>  3     3     3     0 ni    FALSE     FALSE   
#>  4     4     4     0 ni    FALSE     FALSE   
#>  5     5     5     0 ni    FALSE     FALSE   
#>  6     6     6     0 ni    FALSE     FALSE   
#>  7     7     7     0 ni    FALSE     FALSE   
#>  8     8     8     0 ni    FALSE     FALSE   
#>  9     9     9     0 ni    FALSE     FALSE   
#> 10    10    10     0 ni    FALSE     FALSE   
#> # ℹ 622 more rows
#> 
#> $nj
#> # A tibble: 664 × 6
#>       nj index    id name  coord_dim selected
#>    <int> <int> <int> <chr> <lgl>     <lgl>   
#>  1     1     1     1 nj    FALSE     FALSE   
#>  2     2     2     1 nj    FALSE     FALSE   
#>  3     3     3     1 nj    FALSE     FALSE   
#>  4     4     4     1 nj    FALSE     FALSE   
#>  5     5     5     1 nj    FALSE     FALSE   
#>  6     6     6     1 nj    FALSE     FALSE   
#>  7     7     7     1 nj    FALSE     FALSE   
#>  8     8     8     1 nj    FALSE     FALSE   
#>  9     9     9     1 nj    FALSE     FALSE   
#> 10    10    10     1 nj    FALSE     FALSE   
#> # ℹ 654 more rows
#> 
#> $time
#> # A tibble: 1 × 7
#>      time timestamp           index    id name  coord_dim selected
#>     <dbl> <chr>               <int> <int> <chr> <lgl>     <lgl>   
#> 1 1032204 2017-10-02T12:00:00     1     2 time  TRUE      TRUE
```

### Future improvements

- support multiple sources at once for lazy read of a virtual grid
- support groups
- support compound types
- allow a group-by function for a polygon layer, against a pair of
  dimensions to classify cells
- allow a truly DBI and dplyr level of lazy read, with more filter,
  select, mutate and collect idioms
- provide converters to raster format, stars format

[^1]: I.e. it’s not possible to query a file for an arbitrary sparse set
    of values, without constructing a degenerate hyperslab query for
    each point or reading a hyperslab containing cells not in the set.
    Do you know different? Please let me know!
