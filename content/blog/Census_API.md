+++
author = "Aaron Reinke, MPH"
title = "Using the Census API"
date = "2022-09-19"
description = "Learn How to Use the Census API Using R Programming."
tags = [
    "R",
    "API",
    "Census",
    "Data",
    "Mapping"
]
categories = [
    "R",
    "API",
    "Census",
]
series = ["Census Data"]
+++
 
<!--more-->

### Setup

This article walks through how to access the Census API using R. If you've never
coded in R before, you can still follow along.


### Required Packages

The three packages we need are *tidycensus*, *tigris*, and *tidyverse*. *Tidycensus* is the package we will use to access the US Census and gather data, and *tidyverse* is the package we will use to merge and tidy the data.

```(r)
library(tidycensus)
library(tigris)
library(tidyverse)
```

### Census Key

First, we want input our Census API key. You can obtain one through the Census website located here: https://api.census.gov/data/key_signup.html.

```(r)
key <- rstudioapi::askForPassword(prompt = "Please Enter Your API Key")
tidycensus::census_api_key(key = key)
```

### API Call
This example will call the API to fetch Polk County, Iowa data by Census tract. The variables in the example are population counts for Hispanic and non-Hispanic by race. The *summary_var* argument is the total population in each Census Tract. This will be used later to create proportions. The year argument is the *year* of the decennial data I want to pull. Finally, *geometry* is to include the shape files with the API pull.

#### Basic Example

```{r}
PolkCounty_race <- get_decennial(
  geography = "tract",
  state = "IA",
  county = "Polk",
  table = "P2",
  year = 2020,
) 
```
This example above will pull all Census Tracts in Polk County, Iowa using the decennial
Census for the year 2020. The *table* argument specifies which variables the API should pull. This example pulls all the variable related to Hispanic origin by race.

#### More Advanced Example
```{r}
PolkCounty_race <- get_decennial(
  geography = "tract", #Pulls Census Tract data
  state = "IA",        #For Iowa
  county = "Polk",     #In Polk County
  variables = c(
    Black = "P2_006N", #Pulls the variable for Black or African American, non-Hispanic
  ),
  summary_var = "P2_001N", #The total population of Polk County, Iowa
  year = 2020,         #Pulls Census data for the year 2020
  geometry = TRUE      #Pulls geometric shape file for GIS plotting
) 


## You can create another variable by calculating the proportion of the total 

PolkCounty_race <- PolkCounty_race %>%
  mutate(percent = round(100 * (value / summary_value), digits = 1))
```
The example above will pull all Census Tracts in Polk County, Iowa using the decennial
Census for the year 2020. This time we specified variables instead of table. This allows us to specify which variables we want to pull within the table. If you eventually want to plot the data on the map, use the *geometry* argument. The *geometry* argument asks the API to pull the polygon shape file so you don't need to merge the data later. 

### Optional Settings
Downloading shape files is time consuming, so instead of downloading them each time the script is run, you can use the argument *tigris_use_cache = TRUE* to save the files locally.

```{r}
options(tigris_use_cache = TRUE)
```

### Layer Creation

The following code filters the data set by race, so we can create drill down and plot only the Black or African American, non-Hispanic for analysis.


```{r}
Polk.Black.or.African.American <- filter(PolkCounty_race, 
                         variable == "Black")

```


### Creating a Map

The following packages will allow you to plot the Census data on an interactive map.

```{r}
library(tmap)
library(leaflet)
```

Run the code below to set the map view to interactive.

```{r}
tmap_mode("view")
```

I prefer the Esri base-map. You can select the base-map by running the following code.
```{r}
tmap_options(basemaps = c("Esri.WorldTopoMap")) #<<
```

To create your map, you will need to select which data set you want to plot. Then, you will need to specify which variable you want representing the color of the choropleth map.

```{r}
tmap_obj <- tm_shape(Polk.Black.or.African.American) +
  tm_polygons(col = "Percent",
          style = "quantile",
          n = 7,
          palette = "Purples",
          title = "Percent Black or <br/>
          African American<br/>by Census Tract",
          alpha = 0.6,
          group = "Black.or.African.American",
          id = "NAME")
```

### Final Code

```(r)
library(tidycensus)
library(tigris)
library(tidyverse)
library(tmap)
library(leaflet)

key <- rstudioapi::askForPassword(prompt = "Please Enter Your API Key")
tidycensus::census_api_key(key = key)

options(tigris_use_cache = TRUE)

PolkCounty_race <- get_decennial(
  geography = "tract", #Pulls Census Tract data
  state = "IA",        #For Iowa
  county = "Polk",     #In Polk County
  variables = c(
    Black = "P2_006N", #Pulls the variable for Black or African American, non-Hispanic
  ),
  summary_var = "P2_001N", #The total population of Polk County, Iowa
  year = 2020,         #Pulls Census data for the year 2020
  geometry = TRUE      #Pulls geometric shape file for GIS plotting
) 

PolkCounty_race <- PolkCounty_race %>%
  mutate(percent = round(100 * (value / summary_value), digits = 1))
  

###Layer Creation###

Polk.Black.or.African.American <- filter(PolkCounty_race, 
                         variable == "Black")
                         
                         
###Plotting Maps###

tmap_mode("view")
tmap_options(basemaps = c("Esri.WorldTopoMap"))

tmap_obj <- tm_shape(Polk.Black.or.African.American) +
  tm_polygons(col = "Percent",
          style = "quantile",
          n = 7,
          palette = "Purples",
          title = "Percent Black or <br/>
          African American<br/>by Census Tract",
          alpha = 0.6,
          group = "Black.or.African.American",
          id = "NAME")
          
tmap_obj        

```