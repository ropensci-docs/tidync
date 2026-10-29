# tidync: Tidy Approach to 'NetCDF' Data Exploration and Extraction

Tidy tools for 'NetCDF' data sources. Explore the contents of a 'NetCDF'
source (file or URL) presented as variables organized by grid with a
database-like interface. The hyper_filter() interactive function
translates the filter value or index expressions to array-slicing form.
No data is read until explicitly requested, as a data frame or list of
arrays via hyper_tibble() or hyper_array().

Provides easy to use idioms for working with NetCDF data for extraction,
manipulation and visualization. NetCDF is Network Common Data Form
<https://www.unidata.ucar.edu/software/netcdf>.

## Details

See
[`print.tidync()`](https://docs.ropensci.org/tidync/reference/print.tidync.md)
for details on the printed version of a tidync object.

There is a family of functions "hyper_verb" around exploring and
extracting data.

|  |  |
|----|----|
| [`active`](https://docs.ropensci.org/tidync/reference/activate.md) | report the currently active grid |
| [`activate`](https://docs.ropensci.org/tidync/reference/activate.md) | active a grid |
| [`tidync`](https://docs.ropensci.org/tidync/reference/tidync.md) | core NetCDF source object for tidync functions |
| [`hyper_filter`](https://docs.ropensci.org/tidync/reference/hyper_filter.md) | apply dimension expressions to specify array slices |
| [`hyper_array`](https://docs.ropensci.org/tidync/reference/hyper_array.md) | extracts a raw data array based on a NetCDF index |
| [`hyper_tbl_cube`](https://docs.ropensci.org/tidync/reference/hyper_tbl_cube.md) | extracts data as a dplyr tbl_cube |
| [`hyper_tibble`](https://docs.ropensci.org/tidync/reference/hyper_tibble.md) | extracts data as a data frame with all dimension values |
| [`hyper_transforms`](https://docs.ropensci.org/tidync/reference/hyper_transforms.md) | extract the active (or all) dimension transforms |
| [`hyper_vars`](https://docs.ropensci.org/tidync/reference/hyper_vars.md) | information on active variables |
| [`hyper_dims`](https://docs.ropensci.org/tidync/reference/hyper_vars.md) | information on active dimensions |
| [`hyper_grids`](https://docs.ropensci.org/tidync/reference/hyper_vars.md) | information on grids |

The scheme generally processes dimension filters into NetCDF extraction
indexes and these are always available to each function, and are
expressed in printed output.

The following options are available.

|  |  |
|----|----|
| `tidync.large.data.check = TRUE/FALSE` | check for large data extraction (default `TRUE`) |
| `tidync.silent = FALSE/TRUE` | emit warnings,messages or be silent (default `FALSE`) |

## See also

Useful links:

- <https://docs.ropensci.org/tidync/>

- Report bugs at <https://github.com/ropensci/tidync/issues>

## Author

**Maintainer**: Michael Sumner <mdsumner@gmail.com> \[copyright holder\]

Authors:

- Michael Sumner <mdsumner@gmail.com> \[copyright holder\]

Other contributors:

- Simon Wotherspoon \[contributor\]

- Tomas Remenyi \[contributor\]

- Ben Raymond \[contributor\]

- Jakub Nowosad \[contributor\]

- Tim Lucas \[contributor\]

- Hadley Wickham \[contributor\]

- Adrian Odenweller \[contributor\]

- Patrick Van Laake \[contributor\]

- Fabian Bernhard \[contributor\]

## Examples

``` r
argofile <- system.file("extdata/argo/MD5903593_001.nc", package = "tidync")
argo <- tidync(argofile)
argo |> active()
#> [1] "D10,D8"
argo |> activate("D3,D8") |> hyper_array()
#> Class: tidync_data (list of tidync data arrays)
#> Variables (2): 'PLATFORM_NUMBER', 'POSITIONING_SYSTEM'
#> Dimension (1): STRING8,N_PROF (2)
#> Source: /github/home/R/x86_64-pc-linux-gnu-library/4.6/tidync/extdata/argo/MD5903593_001.nc
argo |> hyper_filter(N_LEVELS = index < 4)
#> 
#> Data Source (1): MD5903593_001.nc ...
#> 
#> Grids (16) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D9,D11,D8 : SCIENTIFIC_CALIB_DATE
#> [2]   D6,D9,D11,D8 : PARAMETER
#> [3]   D7,D9,D11,D8 : SCIENTIFIC_CALIB_EQUATION, SCIENTIFIC_CALIB_COEFFICIENT, SCIENTIFIC_CALIB_COMMENT
#> [4]   D6,D9,D8     : STATION_PARAMETERS
#> [5]   D10,D8       : PRES, PRES_QC, PRES_ADJUSTED, PRES_ADJUSTED_QC, PRES_ADJUSTED_ERROR, TEMP, TEMP_QC, TEMP_ADJUSTED, TEMP_ADJUSTED_QC, TEMP_ADJUSTED_ERROR, PSAL, PSAL_QC, PSAL_ADJUSTED, PSAL_ADJUSTED_QC, PSAL_ADJUSTED_ERROR, DOXY, DOXY_QC, DOXY_ADJUSTED, DOXY_ADJUSTED_QC, DOXY_ADJUSTED_ERROR, CHLA, CHLA_QC, CHLA_ADJUSTED, CHLA_ADJUSTED_QC, CHLA_ADJUSTED_ERROR, BBP700, BBP700_QC, BBP700_ADJUSTED, BBP700_ADJUSTED_QC, BBP700_ADJUSTED_ERROR, NITRATE, NITRATE_QC, NITRATE_ADJUSTED, NITRATE_ADJUSTED_QC, NITRATE_ADJUSTED_ERROR    **ACTIVE GRID** ( 986  values per variable)
#> [6]   D1,D8        : DATA_CENTRE
#> [7]   D2,D8        : DATA_STATE_INDICATOR, WMO_INST_TYPE
#> [8]   D3,D8        : PLATFORM_NUMBER, POSITIONING_SYSTEM
#> [9]   D5,D8        : DC_REFERENCE, PLATFORM_TYPE, FLOAT_SERIAL_NO, FIRMWARE_VERSION
#> [10]   D6,D8        : PROJECT_NAME, PI_NAME
#> [11]   D7,D8        : VERTICAL_SAMPLING_SCHEME
#> [12]   D9,D8        : PARAMETER_DATA_MODE
#> [13]   D0           : REFERENCE_DATE_TIME, DATE_CREATION, DATE_UPDATE
#> [14]   D2           : FORMAT_VERSION, HANDBOOK_VERSION
#> [15]   D5           : DATA_TYPE
#> [16]   D8           : CYCLE_NUMBER, DIRECTION, DATA_MODE, JULD, JULD_QC, JULD_LOCATION, LATITUDE, LONGITUDE, POSITION_QC, CONFIG_MISSION_NUMBER, PROFILE_PRES_QC, PROFILE_TEMP_QC, PROFILE_PSAL_QC, PROFILE_DOXY_QC, PROFILE_CHLA_QC, PROFILE_BBP700_QC, PROFILE_NITRATE_QC
#> 
#> Dimensions 14 (2 active): 
#>   
#>   dim   name     length   min   max start count  dmin  dmax unlim coord_dim 
#>   <chr> <chr>     <dbl> <dbl> <dbl> <int> <int> <dbl> <dbl> <lgl> <lgl>     
#> 1 D8    N_PROF        2     1     2     1     2     1     2 FALSE FALSE     
#> 2 D10   N_LEVELS    493     1   493     1     3     1     3 FALSE FALSE     
#>   
#> Inactive dimensions:
#>   
#>    dim   name       length   min   max unlim coord_dim 
#>    <chr> <chr>       <dbl> <dbl> <dbl> <lgl> <lgl>     
#>  1 D0    DATE_TIME      14     1    14 FALSE FALSE     
#>  2 D1    STRING2         2     1     2 FALSE FALSE     
#>  3 D2    STRING4         4     1     4 FALSE FALSE     
#>  4 D3    STRING8         8     1     8 FALSE FALSE     
#>  5 D4    STRING16       16    NA    NA FALSE FALSE     
#>  6 D5    STRING32       32     1    32 FALSE FALSE     
#>  7 D6    STRING64       64     1    64 FALSE FALSE     
#>  8 D7    STRING256     256     1   256 FALSE FALSE     
#>  9 D9    N_PARAM         7     1     7 FALSE FALSE     
#> 10 D11   N_CALIB         1     1     1 FALSE FALSE     
#> 11 D12   N_HISTORY       0    NA    NA TRUE  FALSE     
#> 12 D13   N_VALUES41     41    NA    NA FALSE FALSE     
argo |> hyper_tbl_cube()
#> Source: local array [986 x 2]
#> D: N_LEVELS [int, 493]
#> D: N_PROF [int, 2]
#> M: PRES [dbl[,2]]
#> M: PRES_QC [chr[,2]]
#> M: PRES_ADJUSTED [dbl[,2]]
#> M: PRES_ADJUSTED_QC [chr[,2]]
#> M: PRES_ADJUSTED_ERROR [dbl[,2]]
#> M: TEMP [dbl[,2]]
#> M: TEMP_QC [chr[,2]]
#> M: TEMP_ADJUSTED [dbl[,2]]
#> M: TEMP_ADJUSTED_QC [chr[,2]]
#> M: TEMP_ADJUSTED_ERROR [dbl[,2]]
#> M: PSAL [dbl[,2]]
#> M: PSAL_QC [chr[,2]]
#> M: PSAL_ADJUSTED [dbl[,2]]
#> M: PSAL_ADJUSTED_QC [chr[,2]]
#> M: PSAL_ADJUSTED_ERROR [dbl[,2]]
#> M: DOXY [dbl[,2]]
#> M: DOXY_QC [chr[,2]]
#> M: DOXY_ADJUSTED [dbl[,2]]
#> M: DOXY_ADJUSTED_QC [chr[,2]]
#> M: DOXY_ADJUSTED_ERROR [dbl[,2]]
#> M: CHLA [dbl[,2]]
#> M: CHLA_QC [chr[,2]]
#> M: CHLA_ADJUSTED [dbl[,2]]
#> M: CHLA_ADJUSTED_QC [chr[,2]]
#> M: CHLA_ADJUSTED_ERROR [dbl[,2]]
#> M: BBP700 [dbl[,2]]
#> M: BBP700_QC [chr[,2]]
#> M: BBP700_ADJUSTED [dbl[,2]]
#> M: BBP700_ADJUSTED_QC [chr[,2]]
#> M: BBP700_ADJUSTED_ERROR [dbl[,2]]
#> M: NITRATE [dbl[,2]]
#> M: NITRATE_QC [chr[,2]]
#> M: NITRATE_ADJUSTED [dbl[,2]]
#> M: NITRATE_ADJUSTED_QC [chr[,2]]
#> M: NITRATE_ADJUSTED_ERROR [dbl[,2]]
argo |> hyper_tibble(select_var = c("TEMP_QC"))
#> # A tibble: 986 × 3
#>    TEMP_QC N_LEVELS N_PROF
#>    <chr>      <int>  <int>
#>  1 1              1      1
#>  2 1              2      1
#>  3 1              3      1
#>  4 1              4      1
#>  5 1              5      1
#>  6 1              6      1
#>  7 1              7      1
#>  8 1              8      1
#>  9 1              9      1
#> 10 1             10      1
#> # ℹ 976 more rows
argo |> hyper_transforms()
#> $N_LEVELS
#> # A tibble: 493 × 6
#>    N_LEVELS index    id name     coord_dim selected
#>       <int> <int> <int> <chr>    <lgl>     <lgl>   
#>  1        1     1    10 N_LEVELS FALSE     TRUE    
#>  2        2     2    10 N_LEVELS FALSE     TRUE    
#>  3        3     3    10 N_LEVELS FALSE     TRUE    
#>  4        4     4    10 N_LEVELS FALSE     TRUE    
#>  5        5     5    10 N_LEVELS FALSE     TRUE    
#>  6        6     6    10 N_LEVELS FALSE     TRUE    
#>  7        7     7    10 N_LEVELS FALSE     TRUE    
#>  8        8     8    10 N_LEVELS FALSE     TRUE    
#>  9        9     9    10 N_LEVELS FALSE     TRUE    
#> 10       10    10    10 N_LEVELS FALSE     TRUE    
#> # ℹ 483 more rows
#> 
#> $N_PROF
#> # A tibble: 2 × 6
#>   N_PROF index    id name   coord_dim selected
#>    <int> <int> <int> <chr>  <lgl>     <lgl>   
#> 1      1     1     8 N_PROF FALSE     TRUE    
#> 2      2     2     8 N_PROF FALSE     TRUE    
#> 
argo |> hyper_vars()
#> # A tibble: 35 × 6
#>       id name                type     ndims natts dim_coord
#>    <int> <chr>               <chr>    <int> <int> <lgl>    
#>  1    37 PRES                NC_FLOAT     2    10 FALSE    
#>  2    38 PRES_QC             NC_CHAR      2     3 FALSE    
#>  3    39 PRES_ADJUSTED       NC_FLOAT     2     9 FALSE    
#>  4    40 PRES_ADJUSTED_QC    NC_CHAR      2     3 FALSE    
#>  5    41 PRES_ADJUSTED_ERROR NC_FLOAT     2     6 FALSE    
#>  6    42 TEMP                NC_FLOAT     2     9 FALSE    
#>  7    43 TEMP_QC             NC_CHAR      2     3 FALSE    
#>  8    44 TEMP_ADJUSTED       NC_FLOAT     2     9 FALSE    
#>  9    45 TEMP_ADJUSTED_QC    NC_CHAR      2     3 FALSE    
#> 10    46 TEMP_ADJUSTED_ERROR NC_FLOAT     2     6 FALSE    
#> # ℹ 25 more rows
argo |> hyper_dims()
#> # A tibble: 2 × 7
#>   name     length start count    id unlim coord_dim
#>   <chr>     <dbl> <int> <int> <int> <lgl> <lgl>    
#> 1 N_LEVELS    493     1   493    10 FALSE FALSE    
#> 2 N_PROF        2     1     2     8 FALSE FALSE    
argo |> hyper_grids()
#> # A tibble: 16 × 4
#>    grid         ndims nvars active
#>    <chr>        <int> <int> <lgl> 
#>  1 D0,D9,D11,D8     4     1 FALSE 
#>  2 D6,D9,D11,D8     4     1 FALSE 
#>  3 D7,D9,D11,D8     4     3 FALSE 
#>  4 D6,D9,D8         3     1 FALSE 
#>  5 D10,D8           2    35 TRUE  
#>  6 D1,D8            2     1 FALSE 
#>  7 D2,D8            2     2 FALSE 
#>  8 D3,D8            2     2 FALSE 
#>  9 D5,D8            2     4 FALSE 
#> 10 D6,D8            2     2 FALSE 
#> 11 D7,D8            2     1 FALSE 
#> 12 D9,D8            2     1 FALSE 
#> 13 D0               1     3 FALSE 
#> 14 D2               1     2 FALSE 
#> 15 D5               1     1 FALSE 
#> 16 D8               1    17 FALSE 

## some global options
getOption("tidync.large.data.check")
#> [1] TRUE

getOption("tidync.silent")
#> [1] FALSE
op <- options(tidync.silent = TRUE)
getOption("tidync.silent")
#> [1] TRUE
options(op)
```
