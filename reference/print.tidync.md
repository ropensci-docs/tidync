# Print tidync object

Provide a summary of variables and dimensions, organized by their 'grid'
(or 'shape') and with a summary of any slicing operations provided as
'start' and 'count' summaries for each dimension in the active grid.

## Usage

``` r
# S3 method for class 'tidync'
print(x, ...)
```

## Arguments

- x:

  NetCDF object

- ...:

  reserved

## Value

the input object invisibly

## Details

See [tidync](https://docs.ropensci.org/tidync/reference/tidync.md) for
detail about the object, and
[hyper_vars](https://docs.ropensci.org/tidync/reference/hyper_vars.md)
for programmatic access to the active grid's variable and dimension
information.

The print summary is organized in two sections, the first is available
grids (sets of dimensions) and their associated variables, the second is
the dimensions, separated into active and inactive. All dimensions may
be active in some NetCDF sources.

Individual *active* dimensions include the following components: \*
'dim' - dimension label, D0, D1, D2, ... \* 'name' - dimension name \*
'length' - size of the dimension \* 'min' - minimum value of the
dimension \* 'max' - maximum value of the dimension \* 'start' - start
index of subsetting \* 'count' - length of subsetting index \* 'dmin' -
minimum value of the subset dimension \* 'dmax' - maximum value of the
subset dimension \* 'unlim'

- indicates whether dimension is unlimited (spread across other files,
  usually the time-step) \* 'coord_dim' - indicates whether dimension is
  a coordinate-dimension (i.e. listed as a 1-D grid)

The *inactive* dimension summary does not include 'start', 'count',
'dmin', 'dmax' as these are identical to the values of 1, 'length',
'min', 'max' when no array subsetting has been applied.

## Examples

``` r
argofile <- system.file("extdata/argo/MD5903593_001.nc", package = "tidync")
argo <- tidync(argofile)
print(argo)
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
#> 2 D10   N_LEVELS    493     1   493     1   493     1   493 FALSE FALSE     
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

## the print is modified by choosing a new grid or running filters
argo |> activate("D7,D9,D11,D8")
#> 
#> Data Source (1): MD5903593_001.nc ...
#> 
#> Grids (16) <dimension family> : <associated variables> 
#> 
#> [1]   D0,D9,D11,D8 : SCIENTIFIC_CALIB_DATE
#> [2]   D6,D9,D11,D8 : PARAMETER
#> [3]   D7,D9,D11,D8 : SCIENTIFIC_CALIB_EQUATION, SCIENTIFIC_CALIB_COEFFICIENT, SCIENTIFIC_CALIB_COMMENT    **ACTIVE GRID** ( 3584  values per variable)
#> [4]   D6,D9,D8     : STATION_PARAMETERS
#> [5]   D10,D8       : PRES, PRES_QC, PRES_ADJUSTED, PRES_ADJUSTED_QC, PRES_ADJUSTED_ERROR, TEMP, TEMP_QC, TEMP_ADJUSTED, TEMP_ADJUSTED_QC, TEMP_ADJUSTED_ERROR, PSAL, PSAL_QC, PSAL_ADJUSTED, PSAL_ADJUSTED_QC, PSAL_ADJUSTED_ERROR, DOXY, DOXY_QC, DOXY_ADJUSTED, DOXY_ADJUSTED_QC, DOXY_ADJUSTED_ERROR, CHLA, CHLA_QC, CHLA_ADJUSTED, CHLA_ADJUSTED_QC, CHLA_ADJUSTED_ERROR, BBP700, BBP700_QC, BBP700_ADJUSTED, BBP700_ADJUSTED_QC, BBP700_ADJUSTED_ERROR, NITRATE, NITRATE_QC, NITRATE_ADJUSTED, NITRATE_ADJUSTED_QC, NITRATE_ADJUSTED_ERROR
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
#> Dimensions 14 (4 active): 
#>   
#>   dim   name      length   min   max start count  dmin  dmax unlim coord_dim 
#>   <chr> <chr>      <dbl> <dbl> <dbl> <int> <int> <dbl> <dbl> <lgl> <lgl>     
#> 1 D7    STRING256    256     1   256     1   256     1   256 FALSE FALSE     
#> 2 D8    N_PROF         2     1     2     1     2     1     2 FALSE FALSE     
#> 3 D9    N_PARAM        7     1     7     1     7     1     7 FALSE FALSE     
#> 4 D11   N_CALIB        1     1     1     1     1     1     1 FALSE FALSE     
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
#>  8 D10   N_LEVELS      493     1   493 FALSE FALSE     
#>  9 D12   N_HISTORY       0    NA    NA TRUE  FALSE     
#> 10 D13   N_VALUES41     41    NA    NA FALSE FALSE     

argo |> hyper_filter(N_LEVELS = index > 300)
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
#> 2 D10   N_LEVELS    493     1   493   301   193   301   493 FALSE FALSE     
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
```
