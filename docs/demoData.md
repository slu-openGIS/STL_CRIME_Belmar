Demographic Data
================
Christopher Prener, Ph.D.
(January 15, 2019)

## Introduction

This notebook contains the code for creating neighborhood population
estimates.

## Dependencies

This notebook requires:

``` r
# tidyverse packages
library(dplyr)         # data wrangling
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(readr)         # working with csv data

# spatial packages
library(areal)         # interpolation
library(sf)            # working with spatial data
```

    ## Linking to GEOS 3.6.1, GDAL 2.1.3, PROJ 4.9.3

``` r
library(tidycensus)    # census api access
library(tigris)        # tiger/line api access
```

    ## To enable 
    ## caching of data, set `options(tigris_use_cache = TRUE)` in your R script or .Rprofile.

    ## 
    ## Attaching package: 'tigris'

    ## The following object is masked from 'package:graphics':
    ## 
    ##     plot

``` r
# other packages
library(here)          # file path management
```

    ## here() starts at /Users/chris/GitHub/Personal/STL_CRIME_Belmar

## Download Data

Census data will be used and is accessed via the Census Bureau’s API.
The `tidycensus` and `tigris` packages are used for this.

First, we’ll download a shapefile of the city’s census geography:

``` r
tracts(state = 29, count = 510, class = "sf") %>%
  select(GEOID) -> tracts
```

    ## 
      |                                                                       
      |                                                                 |   0%
      |                                                                       
      |                                                                 |   1%
      |                                                                       
      |=                                                                |   1%
      |                                                                       
      |=                                                                |   2%
      |                                                                       
      |==                                                               |   2%
      |                                                                       
      |==                                                               |   3%
      |                                                                       
      |==                                                               |   4%
      |                                                                       
      |===                                                              |   4%
      |                                                                       
      |===                                                              |   5%
      |                                                                       
      |====                                                             |   5%
      |                                                                       
      |====                                                             |   6%
      |                                                                       
      |====                                                             |   7%
      |                                                                       
      |=====                                                            |   7%
      |                                                                       
      |=====                                                            |   8%
      |                                                                       
      |======                                                           |   9%
      |                                                                       
      |======                                                           |  10%
      |                                                                       
      |=======                                                          |  10%
      |                                                                       
      |=======                                                          |  11%
      |                                                                       
      |========                                                         |  12%
      |                                                                       
      |========                                                         |  13%
      |                                                                       
      |=========                                                        |  13%
      |                                                                       
      |=========                                                        |  14%
      |                                                                       
      |=========                                                        |  15%
      |                                                                       
      |==========                                                       |  15%
      |                                                                       
      |==========                                                       |  16%
      |                                                                       
      |===========                                                      |  16%
      |                                                                       
      |===========                                                      |  17%
      |                                                                       
      |===========                                                      |  18%
      |                                                                       
      |============                                                     |  18%
      |                                                                       
      |============                                                     |  19%
      |                                                                       
      |=============                                                    |  19%
      |                                                                       
      |=============                                                    |  20%
      |                                                                       
      |=============                                                    |  21%
      |                                                                       
      |==============                                                   |  21%
      |                                                                       
      |==============                                                   |  22%
      |                                                                       
      |===============                                                  |  22%
      |                                                                       
      |===============                                                  |  23%
      |                                                                       
      |===============                                                  |  24%
      |                                                                       
      |================                                                 |  24%
      |                                                                       
      |================                                                 |  25%
      |                                                                       
      |=================                                                |  25%
      |                                                                       
      |=================                                                |  26%
      |                                                                       
      |=================                                                |  27%
      |                                                                       
      |==================                                               |  27%
      |                                                                       
      |==================                                               |  28%
      |                                                                       
      |===================                                              |  29%
      |                                                                       
      |===================                                              |  30%
      |                                                                       
      |====================                                             |  30%
      |                                                                       
      |====================                                             |  31%
      |                                                                       
      |=====================                                            |  32%
      |                                                                       
      |=====================                                            |  33%
      |                                                                       
      |======================                                           |  33%
      |                                                                       
      |======================                                           |  34%
      |                                                                       
      |======================                                           |  35%
      |                                                                       
      |=======================                                          |  35%
      |                                                                       
      |=======================                                          |  36%
      |                                                                       
      |========================                                         |  36%
      |                                                                       
      |========================                                         |  37%
      |                                                                       
      |========================                                         |  38%
      |                                                                       
      |=========================                                        |  38%
      |                                                                       
      |=========================                                        |  39%
      |                                                                       
      |==========================                                       |  39%
      |                                                                       
      |==========================                                       |  40%
      |                                                                       
      |==========================                                       |  41%
      |                                                                       
      |===========================                                      |  41%
      |                                                                       
      |===========================                                      |  42%
      |                                                                       
      |============================                                     |  42%
      |                                                                       
      |============================                                     |  43%
      |                                                                       
      |============================                                     |  44%
      |                                                                       
      |=============================                                    |  44%
      |                                                                       
      |=============================                                    |  45%
      |                                                                       
      |==============================                                   |  46%
      |                                                                       
      |==============================                                   |  47%
      |                                                                       
      |===============================                                  |  47%
      |                                                                       
      |===============================                                  |  48%
      |                                                                       
      |================================                                 |  48%
      |                                                                       
      |================================                                 |  49%
      |                                                                       
      |================================                                 |  50%
      |                                                                       
      |=================================                                |  50%
      |                                                                       
      |=================================                                |  51%
      |                                                                       
      |==================================                               |  52%
      |                                                                       
      |==================================                               |  53%
      |                                                                       
      |===================================                              |  53%
      |                                                                       
      |===================================                              |  54%
      |                                                                       
      |===================================                              |  55%
      |                                                                       
      |====================================                             |  55%
      |                                                                       
      |====================================                             |  56%
      |                                                                       
      |=====================================                            |  56%
      |                                                                       
      |=====================================                            |  57%
      |                                                                       
      |=====================================                            |  58%
      |                                                                       
      |======================================                           |  58%
      |                                                                       
      |======================================                           |  59%
      |                                                                       
      |=======================================                          |  59%
      |                                                                       
      |=======================================                          |  60%
      |                                                                       
      |=======================================                          |  61%
      |                                                                       
      |========================================                         |  61%
      |                                                                       
      |========================================                         |  62%
      |                                                                       
      |=========================================                        |  62%
      |                                                                       
      |=========================================                        |  63%
      |                                                                       
      |=========================================                        |  64%
      |                                                                       
      |==========================================                       |  64%
      |                                                                       
      |==========================================                       |  65%
      |                                                                       
      |===========================================                      |  65%
      |                                                                       
      |===========================================                      |  66%
      |                                                                       
      |===========================================                      |  67%
      |                                                                       
      |============================================                     |  67%
      |                                                                       
      |============================================                     |  68%
      |                                                                       
      |=============================================                    |  69%
      |                                                                       
      |=============================================                    |  70%
      |                                                                       
      |==============================================                   |  70%
      |                                                                       
      |==============================================                   |  71%
      |                                                                       
      |===============================================                  |  72%
      |                                                                       
      |===============================================                  |  73%
      |                                                                       
      |================================================                 |  73%
      |                                                                       
      |================================================                 |  74%
      |                                                                       
      |================================================                 |  75%
      |                                                                       
      |=================================================                |  75%
      |                                                                       
      |=================================================                |  76%
      |                                                                       
      |==================================================               |  76%
      |                                                                       
      |==================================================               |  77%
      |                                                                       
      |==================================================               |  78%
      |                                                                       
      |===================================================              |  78%
      |                                                                       
      |===================================================              |  79%
      |                                                                       
      |====================================================             |  79%
      |                                                                       
      |====================================================             |  80%
      |                                                                       
      |====================================================             |  81%
      |                                                                       
      |=====================================================            |  81%
      |                                                                       
      |=====================================================            |  82%
      |                                                                       
      |======================================================           |  82%
      |                                                                       
      |======================================================           |  83%
      |                                                                       
      |======================================================           |  84%
      |                                                                       
      |=======================================================          |  84%
      |                                                                       
      |=======================================================          |  85%
      |                                                                       
      |========================================================         |  86%
      |                                                                       
      |========================================================         |  87%
      |                                                                       
      |=========================================================        |  87%
      |                                                                       
      |=========================================================        |  88%
      |                                                                       
      |==========================================================       |  88%
      |                                                                       
      |==========================================================       |  89%
      |                                                                       
      |==========================================================       |  90%
      |                                                                       
      |===========================================================      |  90%
      |                                                                       
      |===========================================================      |  91%
      |                                                                       
      |============================================================     |  92%
      |                                                                       
      |============================================================     |  93%
      |                                                                       
      |=============================================================    |  93%
      |                                                                       
      |=============================================================    |  94%
      |                                                                       
      |==============================================================   |  95%
      |                                                                       
      |==============================================================   |  96%
      |                                                                       
      |===============================================================  |  96%
      |                                                                       
      |===============================================================  |  97%
      |                                                                       
      |===============================================================  |  98%
      |                                                                       
      |================================================================ |  98%
      |                                                                       
      |================================================================ |  99%
      |                                                                       
      |=================================================================|  99%
      |                                                                       
      |=================================================================| 100%

Next, we’ll download the 2017 five year population estimates, and merge
them with the census geography data we’ve already cleaned above:

``` r
# 2017
get_acs(year = 2017, geography = "tract", variable = "B01003_001", 
        state = 29, county = 510) %>%
  left_join(tracts, ., by = "GEOID") %>%
  st_transform(crs = 102696) -> tracts17
```

    ## Getting data from the 2013-2017 5-year ACS

## Load Neighborhood Data

To estimate populations for neighborhoods, we’ll use the city’s
neighborhood boundary
data:

``` r
st_read(here("data", "raw", "nhood", "STL_BOUNDARY_Nhoods.shp"), stringsAsFactors = FALSE) %>%
  st_transform(crs = 102696) %>%
  select(NHD_NUM, NHD_NAME) -> nhoods
```

    ## Reading layer `STL_BOUNDARY_Nhoods' from data source `/Users/chris/GitHub/Personal/STL_CRIME_Belmar/data/raw/nhood/STL_BOUNDARY_Nhoods.shp' using driver `ESRI Shapefile'
    ## Simple feature collection with 88 features and 6 fields
    ## geometry type:  MULTIPOLYGON
    ## dimension:      XY
    ## bbox:           xmin: 871512.3 ymin: 982994.4 xmax: 912850.5 ymax: 1070957
    ## epsg (SRID):    NA
    ## proj4string:    +proj=tmerc +lat_0=35.83333333333334 +lon_0=-90.5 +k=0.9999333333333333 +x_0=250000 +y_0=0 +datum=NAD83 +units=us-ft +no_defs

## Estimate Neighborhood Populations

We’ll use a technique called areal weighted interpolation to estimate
populations for each neighborhood.

``` r
aw_interpolate(nhoods, tid = NHD_NUM, source = tracts17, sid = GEOID, 
               weight = "sum", output = "tibble", extensive = "estimate") %>%
  left_join(nhoods, ., by = "NHD_NUM") %>%
  rename(pop17 = estimate) -> nhoods
```

## Clean-up Enviornment

We can get rid of the `tracts` and `tracts17` objects at this stage.

``` r
rm(tracts, tracts17)
```

## Add Part 1 Crime Counts by Neighborhood

Next, we want to add counts of crimes for each neighborhood. To do this,
we need to load the crime data we cleaned previously:

``` r
p1_crimes <- read_csv(here("data", "clean", "p1_crimes.csv"))
```

    ## Parsed with column specification:
    ## cols(
    ##   Complaint = col_character(),
    ##   DateOccur = col_character(),
    ##   Crime = col_double(),
    ##   crimeCat = col_character(),
    ##   Description = col_character(),
    ##   ILEADSAddress = col_double(),
    ##   ILEADSStreet = col_character(),
    ##   XCoord = col_double(),
    ##   YCoord = col_double(),
    ##   xyStatus = col_logical()
    ## )

Then, we need to project the data, place them in the appropriate
coordinate system, and perform a spatial join to identify the
neighborhood each crime occurs in:

``` r
# obtain counts by neighborhood
p1_crimes %>%
  st_as_sf(coords = c("XCoord", "YCoord"), crs = 102696) %>%
  st_intersection(., nhoods) %>%
  group_by(NHD_NUM) %>%
  summarise(p1Crimes = n()) -> nhoodCrimes
```

    ## Warning: attribute variables are assumed to be spatially constant
    ## throughout all geometries

``` r
# remove geometry
st_geometry(nhoodCrimes) <- NULL

# join with original neighborhood data
nhoods <- left_join(nhoods, nhoodCrimes, by = "NHD_NUM")
```

Since crime rates are the most important metric, we’ll add a measure of
crimes per 1,000 residents in each neighborhood:

``` r
nhoods <- mutate(nhoods, p1Rate = (p1Crimes/pop17)*1000)
```

## Add Violent Crime Counts by Neighborhood

Next, we want to add counts of violent crimes for each neighborhood. To
do this, we need to load the crime data we cleaned previously:

``` r
v_crimes <- read_csv(here("data", "clean", "v_crimes.csv"))
```

    ## Parsed with column specification:
    ## cols(
    ##   Complaint = col_character(),
    ##   DateOccur = col_character(),
    ##   Crime = col_double(),
    ##   crimeCat = col_character(),
    ##   Description = col_character(),
    ##   ILEADSAddress = col_double(),
    ##   ILEADSStreet = col_character(),
    ##   XCoord = col_double(),
    ##   YCoord = col_double(),
    ##   xyStatus = col_logical()
    ## )

Then, we need to project the data, place them in the appropriate
coordinate system, and perform a spatial join to identify the
neighborhood each crime occurs in:

``` r
# obtain counts by neighborhood
v_crimes %>%
  st_as_sf(coords = c("XCoord", "YCoord"), crs = 102696) %>%
  st_intersection(., nhoods) %>%
  group_by(NHD_NUM) %>%
  summarise(vCrimes = n()) -> nhoodVCrimes
```

    ## Warning: attribute variables are assumed to be spatially constant
    ## throughout all geometries

``` r
# remove geometry
st_geometry(nhoodVCrimes) <- NULL

# join with original neighborhood data
left_join(nhoods, nhoodVCrimes, by = "NHD_NUM") %>%
  mutate(vCrimes = ifelse(is.na(vCrimes) == TRUE, 0, vCrimes)) -> nhoods
```

Since crime rates are the most important metric, we’ll add a measure of
crimes per 1,000 residents in each neighborhood:

``` r
nhoods <- mutate(nhoods, vRate = (vCrimes/pop17)*1000)
```

## Write Data

Finally, we’ll write our neighborhood data to a shapefile for mapping:

``` r
st_write(nhoods, 
         here("data", "clean", "STL_CRIME_Neighborhoods", "STL_CRIME_Neighborhoods.shp"),
         delete_dsn = TRUE)
```

    ## Deleting source `/Users/chris/GitHub/Personal/STL_CRIME_Belmar/data/clean/STL_CRIME_Neighborhoods/STL_CRIME_Neighborhoods.shp' using driver `ESRI Shapefile'
    ## Writing layer `STL_CRIME_Neighborhoods' to data source `/Users/chris/GitHub/Personal/STL_CRIME_Belmar/data/clean/STL_CRIME_Neighborhoods/STL_CRIME_Neighborhoods.shp' using driver `ESRI Shapefile'
    ## features:       88
    ## fields:         7
    ## geometry type:  Multi Polygon
