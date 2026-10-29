# Helper to get a variable from NetCDF.

This exists so we can (internally) use a file path, uri, or open NetCDF
connection (ncdf4 or RNetCDF) in a simpler way.

## Usage

``` r
nc_get(x, v, test = FALSE)
```

## Arguments

- x:

  file path, uri, or NetCDF connection

- v:

  variable name

- test:

  if true we make sure the connection can be open, not applied for
  connections themselves

## Value

array of values for the requested variable

## Details

This function just reads the whole array. It is equivalent to the
angstroms package function 'rawdata(x, varname)'.

## Examples

``` r
l3file <- "S20080012008031.L3m_MO_CHL_chlor_a_9km.nc"
fpath <- system.file("extdata", "oceandata", l3file,
package = "tidync")
lat <- nc_get(fpath, "lat")
```
