Data Creation
================
Christopher Prener, Ph.D.
(January 15, 2019)

## Introduction

This notebook contains the code for cleaning the 2016-2017 crime file
data.

## Dependencies

This notebook requires:

``` r
# primary data tools
library(compstatr)     # work with stlmpd crime data

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

# other packages
library(janitor)       # frequency tables
library(here)          # file path management
```

    ## here() starts at /Users/chris/GitHub/Personal/STL_CRIME_Belmar

``` r
library(knitr)         # output
library(testthat)      # unit testing
```

    ## 
    ## Attaching package: 'testthat'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     matches

## Create Data

Data downloaded from the STLMPD website come in `.csv` format but with
the wrong file extension. The following bash script copies them to a new
subdirectory and fixes the file extension issue:

``` bash
# change working directory
cd ..
# execute cleaning script
bash source/reformatHTML.sh
```

    ## mkdir: data/intermediate/2016: File exists
    ## mkdir: data/intermediate/2017: File exists
    ## mkdir: data/intermediate/2018: File exists

## Load Data

With our data renamed, we build a year list objects for 2016 and 2017
crimes:

``` r
data2016 <- cs_load_year(here("data", "intermediate", "2016"))
data2017 <- cs_load_year(here("data", "intermediate", "2017"))
data2018 <- cs_load_year(here("data", "intermediate", "2018"))
```

## Validate Data

The data validation process is currently broken in `compstatr` due to
changes in how SLMPD is shipping their data. We’ll get these data to a
minimum level of validation before proceeding, but they won’t fully pass
`compstatr`’s tests.

### 2016

Next we make sure there are no problems with the crime files in terms of
incongruent columns for 2015:

``` r
cs_validate_year(data2016, year = "2016")
```

    ## [1] FALSE

We can use the `verbose = TRUE` option on `cs_validate_year()` to
identify areas where the validation checks have failed:

``` r
cs_validate_year(data2016, year = "2016", verbose = TRUE)
```

    ## # A tibble: 12 x 9
    ##    namedMonth codedMonth valMonth codedYear valYear oneMonth varCount
    ##    <chr>      <chr>      <lgl>        <int> <lgl>   <lgl>    <lgl>   
    ##  1 January    January    TRUE          2016 TRUE    TRUE     TRUE    
    ##  2 February   February   TRUE          2016 TRUE    TRUE     TRUE    
    ##  3 March      March      TRUE          2016 TRUE    TRUE     TRUE    
    ##  4 April      April      TRUE          2016 TRUE    TRUE     TRUE    
    ##  5 May        May        TRUE          2016 TRUE    TRUE     TRUE    
    ##  6 June       June       TRUE          2016 TRUE    TRUE     TRUE    
    ##  7 July       July       TRUE          2016 TRUE    TRUE     TRUE    
    ##  8 August     August     TRUE          2016 TRUE    TRUE     TRUE    
    ##  9 September  September  TRUE          2016 TRUE    TRUE     TRUE    
    ## 10 October    October    TRUE          2016 TRUE    TRUE     TRUE    
    ## 11 November   November   TRUE          2016 TRUE    TRUE     TRUE    
    ## 12 December   December   TRUE          2016 TRUE    TRUE     TRUE    
    ## # … with 2 more variables: valVars <lgl>, valClasses <lgl>

Since there are only issues with the class validation, we’re going to
consider these validated for now.

### 2017

We’ll repeat the same validation for 2017:

``` r
cs_validate_year(data2017, year = "2017")
```

    ## [1] FALSE

We can use the `verbose = TRUE` option on `cs_validate_year()` to
identify areas where the validation checks have failed:

``` r
cs_validate_year(data2017, year = "2017", verbose = TRUE)
```

    ## # A tibble: 12 x 9
    ##    namedMonth codedMonth valMonth codedYear valYear oneMonth varCount
    ##    <chr>      <chr>      <lgl>        <int> <lgl>   <lgl>    <lgl>   
    ##  1 January    January    TRUE          2017 TRUE    TRUE     TRUE    
    ##  2 February   February   TRUE          2017 TRUE    TRUE     TRUE    
    ##  3 March      March      TRUE          2017 TRUE    TRUE     TRUE    
    ##  4 April      April      TRUE          2017 TRUE    TRUE     TRUE    
    ##  5 May        May        TRUE          2017 TRUE    TRUE     FALSE   
    ##  6 June       June       TRUE          2017 TRUE    TRUE     TRUE    
    ##  7 July       July       TRUE          2017 TRUE    TRUE     TRUE    
    ##  8 August     August     TRUE          2017 TRUE    TRUE     TRUE    
    ##  9 September  September  TRUE          2017 TRUE    TRUE     TRUE    
    ## 10 October    October    TRUE          2017 TRUE    TRUE     TRUE    
    ## 11 November   November   TRUE          2017 TRUE    TRUE     TRUE    
    ## 12 December   December   TRUE          2017 TRUE    TRUE     TRUE    
    ## # … with 2 more variables: valVars <lgl>, valClasses <lgl>

The data for May 2017 do not pass the validation checks. We can extract
this month and confirm that there are too many columns in the May 2017
release. Once we have that confirmed, we can standardize that month and
re-run our validation.

``` r
# extract data
may2017 <- cs_extract_month(data2017, month = "May")
# unit test column number
expect_equal(ncol(may2017), 26)
# remove object
rm(may2017)
# standardize months
data2017 <- cs_standardize(data2017, month = "May", config = 26)
# validate data
cs_validate_year(data2017, year = "2017")
```

    ## [1] FALSE

We still get a `FALSE` value for `cs_validate_year()`:

``` r
cs_validate_year(data2017, year = "2017", verbose = TRUE)
```

    ## # A tibble: 12 x 9
    ##    namedMonth codedMonth valMonth codedYear valYear oneMonth varCount
    ##    <chr>      <chr>      <lgl>        <int> <lgl>   <lgl>    <lgl>   
    ##  1 January    January    TRUE          2017 TRUE    TRUE     TRUE    
    ##  2 February   February   TRUE          2017 TRUE    TRUE     TRUE    
    ##  3 March      March      TRUE          2017 TRUE    TRUE     TRUE    
    ##  4 April      April      TRUE          2017 TRUE    TRUE     TRUE    
    ##  5 May        May        TRUE          2017 TRUE    TRUE     TRUE    
    ##  6 June       June       TRUE          2017 TRUE    TRUE     TRUE    
    ##  7 July       July       TRUE          2017 TRUE    TRUE     TRUE    
    ##  8 August     August     TRUE          2017 TRUE    TRUE     TRUE    
    ##  9 September  September  TRUE          2017 TRUE    TRUE     TRUE    
    ## 10 October    October    TRUE          2017 TRUE    TRUE     TRUE    
    ## 11 November   November   TRUE          2017 TRUE    TRUE     TRUE    
    ## 12 December   December   TRUE          2017 TRUE    TRUE     TRUE    
    ## # … with 2 more variables: valVars <lgl>, valClasses <lgl>

We’ve now limited the validation issues to the classes.

### 2018

Finally, we’ll validate the 2018 data, which contain some reported
crimes for 2016 and 2017 as well:

``` r
cs_validate_year(data2018, year = "2018")
```

    ## [1] FALSE

Now with the `verbose = TRUE` option:

``` r
cs_validate_year(data2018, year = "2018", verbose = TRUE)
```

    ## # A tibble: 12 x 9
    ##    namedMonth codedMonth valMonth codedYear valYear oneMonth varCount
    ##    <chr>      <chr>      <lgl>        <int> <lgl>   <lgl>    <lgl>   
    ##  1 January    January    TRUE          2018 TRUE    TRUE     TRUE    
    ##  2 February   February   TRUE          2018 TRUE    TRUE     TRUE    
    ##  3 March      March      TRUE          2018 TRUE    TRUE     TRUE    
    ##  4 April      April      TRUE          2018 TRUE    TRUE     TRUE    
    ##  5 May        May        TRUE          2018 TRUE    TRUE     TRUE    
    ##  6 June       June       TRUE          2018 TRUE    TRUE     TRUE    
    ##  7 July       July       TRUE          2018 TRUE    TRUE     TRUE    
    ##  8 August     August     TRUE          2018 TRUE    TRUE     TRUE    
    ##  9 September  September  TRUE          2018 TRUE    TRUE     TRUE    
    ## 10 October    October    TRUE          2018 TRUE    TRUE     TRUE    
    ## 11 November   November   TRUE          2018 TRUE    TRUE     TRUE    
    ## 12 December   December   TRUE          2018 TRUE    TRUE     TRUE    
    ## # … with 2 more variables: valVars <lgl>, valClasses <lgl>

We only have issues with the class validation.

## Collapse Data

With the data validated, we collapse each year into a single, flat
object:

``` r
data2016_flat <- cs_collapse(data2016)
data2017_flat <- cs_collapse(data2017)
data2018_flat <- cs_collapse(data2018)
```

What we need for this project is a single object with only the crimes
for 2016. Since crimes were *reported* in subsequent years for 2015 (as
well as 2016 and 2017), we need to merge all the tables and then retain
only the relevant year’s data. The `cs_combine()` function will do
this:

``` r
crimes2016 <- cs_combine(type = "year", date = 2016, data2016_flat, data2017_flat, data2018_flat)
crimes2017 <- cs_combine(type = "year", date = 2017, data2017_flat, data2018_flat)
```

### Clean-up Environment

With our data created, we can remove some of the intermediary objects
we’ve
created:

``` r
rm(data2016, data2016_flat, data2017, data2017_flat, data2018, data2018_flat)
```

## Remove Unfounded Crimes and Subset Based on Type of Crime:

The following code chunk removes unfounded crimes (those where `Count ==
-1`) and then creates a data frame for all part one crimes for each
year. We also print the number of crimes missing spatial data. In
general, these tend to be rapes. We’re focusing on Part 1 Crimes, which
are defined by the FBI as all violent crimes (aggravated assault, rape,
murder, and robbery) as well as property crimes (arson, burglary,
larceny-theft, and motor vehicle theft).

### 2016

Our initial task to is subset the data. We also add a column
categorizing each crime, and select only the columns we need for this
mapping project:

``` r
crimes2016 %>% 
  cs_filter_count(var = Count) %>%
  cs_filter_crime(var = Crime, crime = "Part 1") %>%
  cs_crime_cat(var = Crime, newVar = crimeCat, output = "string") %>%
  cs_missing_xy(varx = XCoord, vary = YCoord, newVar = xyStatus) %>%
  select(Complaint, DateOccur, Crime, crimeCat, Description, 
         ILEADSAddress, ILEADSStreet, XCoord, YCoord, xyStatus) -> p1_2016
```

Next, we’ll print the number of crimes that are missing spatial data:

``` r
p1_2016 %>%
  tabyl(xyStatus) %>%
  adorn_pct_formatting(digits = 3) %>%
  kable()
```

| xyStatus |     n | percent |
| :------- | ----: | :------ |
| FALSE    | 24954 | 98.078% |
| TRUE     |   489 | 1.922%  |

Finally, we’ll summarize the crimes so that we get a sense of how much
crime falls into specific categories:

``` r
p1_2016 %>%
  tabyl(crimeCat) %>%
  adorn_pct_formatting(digits = 3) %>%
  kable()
```

| crimeCat            |     n | percent |
| :------------------ | ----: | :------ |
| Aggravated Assault  |  3668 | 14.417% |
| Arson               |   297 | 1.167%  |
| Burgalry            |  3270 | 12.852% |
| Forcible Rape       |   281 | 1.104%  |
| Homicide            |   201 | 0.790%  |
| Larceny             | 12511 | 49.173% |
| Motor Vehicle Theft |  3275 | 12.872% |
| Robbery             |  1940 | 7.625%  |

### 2017

For 2017, we’ll again subset and clean the data:

``` r
crimes2017 %>% 
  cs_filter_count(var = Count) %>%
  cs_filter_crime(var = Crime, crime = "Part 1") %>%
  cs_crime_cat(var = Crime, newVar = crimeCat, output = "string") %>%
  cs_missing_xy(varx = XCoord, vary = YCoord, newVar = xyStatus) %>%
  select(Complaint, DateOccur, Crime, crimeCat, Description, 
         ILEADSAddress, ILEADSStreet, XCoord, YCoord, xyStatus) -> p1_2017
```

Next, we’ll print the number of crimes that are missing spatial data:

``` r
p1_2017 %>%
  tabyl(xyStatus) %>%
  adorn_pct_formatting(digits = 3) %>%
  kable()
```

| xyStatus |     n | percent |
| :------- | ----: | :------ |
| FALSE    | 25367 | 98.033% |
| TRUE     |   509 | 1.967%  |

Finally, we’ll summarize the crimes so that we get a sense of how much
crime falls into specific categories:

``` r
p1_2017 %>%
  tabyl(crimeCat) %>%
  adorn_pct_formatting(digits = 3) %>%
  kable()
```

| crimeCat            |     n | percent |
| :------------------ | ----: | :------ |
| Aggravated Assault  |  4074 | 15.744% |
| Arson               |   230 | 0.889%  |
| Burgalry            |  3203 | 12.378% |
| Forcible Rape       |   244 | 0.943%  |
| Homicide            |   220 | 0.850%  |
| Larceny             | 12999 | 50.236% |
| Motor Vehicle Theft |  2919 | 11.281% |
| Robbery             |  1987 | 7.679%  |

## Clean Environment

We no longer need the full crime objects, so we’ll get rid of those now:

``` r
rm(crimes2016, crimes2017)
```

## Combine Objects

Since the Belmar proposal mapped the data for two years, we need to bind
our two data objects together.

``` r
p1_crimes <- bind_rows(p1_2016, p1_2017)
```

## Write Data

Finally, we’ll creat a spreadsheet of our data for later reference:

``` r
write_csv(p1_crimes, here("data", "clean", "p1_crimes.csv"))
```

## Isolate Violent Crimes

We’ll also pull out violent cimes, and write them to a spreadsheet as
well:

``` r
v_crimes <- cs_filter_crime(p1_crimes, var = Crime, crime = "Violent")
write_csv(v_crimes, here("data", "clean", "v_crimes.csv"))
```
