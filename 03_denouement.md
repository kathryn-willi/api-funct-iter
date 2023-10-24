Lesson 3.3: USGS Streamflow
================
Katie Willi
2023-01-27

## Lesson Objectives:

In this lesson you will take all of the skills you have learned up to
this point and use them on a completely new set of data. This lesson has
**five exercises** that need to be completed.

#### Necessary packages:

``` r
library(tidyverse)
library(plotly)
library(scales)
library(httr)
library(jsonlite)
library(dataRetrieval)
library(sf) # for the map
library(mapview) # for making the interactive plot
```

## Streaflow Datasets

We are interested in looking at how the Cache la Poudre River’s flow
changes as it travels out of the mountainous Poudre Canyon and through
Fort Collins.

There are four stream flow monitoring sites on the Poudre that we are
interested in: two managed by the US Geological Survey (USGS), and two
managed by the Colorado Division of Water Resources (CDWR):

![](03_denouement_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

### USGS `dataRetrieval` R package

To pull data for USGS stream gages, we can use the `dataRetrieval`
package, which is a USGS-managed set of functions that, much like our
functions from Lesson 3.1, pull data from the USGS’s data warehouse
using an API. Here we will pull flow data for our USGS stream gages of
interest for the last two water years:

``` r
# pulls USGS daily ('dv') stream flow data:
usgs <- dataRetrieval::readNWISdv(siteNumbers = c("06752260", "06752280"), # USGS site code for the Poudre River at the Lincoln Bridge and the ELC
                               parameterCd = "00060", # USGS code for stream flow
                               startDate = "2020-10-01", # YYYY-MM-DD formatting
                               endDate = "2022-09-30") %>% # YYYY-MM-DD formatting
  rename(q_cfs = X_00060_00003) %>% # USGS code for stream flow units in cubic feet per second (CFS)
  mutate(Date = lubridate::ymd(Date), # convert the Date column to "Date" formatting using the `lubridate` package
         Site = case_when(site_no == "06752260" ~ "Lincoln", 
                          site_no == "06752280" ~ "Boxelder"))
```

### CDWR’s API

Alas, CDWR does NOT have an R package that pulls data from [their
API](https://dwr.state.co.us/Rest/GET/Help#Datasets&#SurfaceWaterController&#gettingstarted&#jsonxml),
but they do have user-friendly directions on how to develop API calls.

Using the “URL generator” steps outlined for their [daily surface water
time series data
set](https://dwr.state.co.us/Rest/GET/Help/SurfaceWaterTSDayGenerator),
we can get the last two water years of CFS data for the Poudre at the
Canyon mouth (site abbreviation = CLAFTCCO) using the following URL:

<https://dwr.state.co.us/Rest/GET/api/v2/surfacewater/surfacewatertsday/?format=json&dateFormat=dateOnly&fields=abbrev%2CmeasDate%2Cvalue%2CmeasUnit&encoding=deflate&abbrev=CLAFTCCO&min-measDate=10%2F01%2F2020&max-measDate=09%2F30%2F2022>

# Exercise \#1

Using the URL above as the starting point, develop a function that
creates a data frame of CDWR daily flow (CFS) data for a selected range
of water years, for any site. (HINT: The final product of our API pull
is a list with additional metadata about our API pull… how do we index a
list to extract the time series flow data?)

# Exercise \#2

Map over the function you developed in Exercise \#1 to pull flow data
for CLAFTCCO and CLARIVCO for the 2021 and 2022 water years.

# Exercise \#3

Join our USGS and CDWR data frames together (`bind_rows()`, perhaps?),
then create an interactive ggplot of discharge (in CFS) through time
displaying all four of our monitoring sites. Be sure all axes and labels
are clear.

# Exercise \#4

Create an interactive plot of the daily difference in discharge between
the Cache la Poudre River at the canyon mouth and each of the sites
downstream. Make sure your plot axes are clear.

# Exercise \#5

For each of our downstream locations, calculate how many days the canyon
mouth had LOWER flow. Is this what you expected? Why or why not?