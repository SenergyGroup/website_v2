+++
author = "Aaron Reinke, MPH"
title = "Using the Census API"
date = "2022-09-19"
description = "Learn How to Use the Census API Using R."
tags = [
    "API",
    "Census",
    "Data",
    "Mapping"
]
categories = [
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

The four packages we need are *tidycensus*, *tigris*, *tmap*,and *tidyverse*. *Tidycensus* is the package we will use to access the US Census and gather data, and *tidyverse* is the package we will use to merge and tidy the data.

```(r)
library(tidycensus)
library(tigris)
library(tmap)
library(leaflet)
library(tidyverse)
```

### Census Key

First, we want input our Census API key. You can obtain one through the Census website located here: https://api.census.gov/data/key_signup.html.

```(r)
key <- rstudioapi::askForPassword(prompt = "Please Enter Your API Key")
tidycensus::census_api_key(key = key)
```

### Optional Settings
Downloading shape files is time consuming, so instead of downloading them each time the script is run, you can use the argument *tigris_use_cache = TRUE* to save the files locally.

```{r}
options(tigris_use_cache = TRUE)
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
The example above will pull all Census Tracts in Polk County, Iowa using the decennial
Census for the year 2020. The *table* argument specifies which variables the API should pull. This example pulls all the variable related to Hispanic origin by race.

#### More Advanced Example
```{r}
PolkCounty_race <- get_decennial(
  geography = "tract",
  state = "IA",
  county = "Polk",
  variables = c(
    Hispanic = "P2_002N",
    White = "P2_005N",
    Black = "P2_006N",
    Native = "P2_007N",
    Asian = "P2_008N",
    Two =  "P2_011N"
  ),
  summary_var = "P2_001N",
  year = 2020,
  geometry = TRUE
) %>%
  mutate(percent = round(100 * (value / summary_value), digits = 1))
```
The example above will pull all Census Tracts in Polk County, Iowa using the decennial
Census for the year 2020. The data pull specifies which variables we want to pull within the P2 table and renames them. The *geometry* argument asks the API to also pull the polygon shape file for plotting using maps. 

### Layer Creation

The following code filters the dataset by each race and ethnicity so we can create separate shape files for analysis.


```{r}
Polk.Black.or.African.American <- filter(blackhawk_race, 
                         variable == "Black")
Polk.Hispanic.or.Latino <- filter(blackhawk_race, 
                         variable == "Hispanic")
Polk.White <- filter(blackhawk_race, 
                         variable == "White")
Polk.Native.American.or.Alaska.Native <- filter(blackhawk_race, 
                         variable == "Native")
Polk.Asian <- filter(blackhawk_race, 
                         variable == "Asian")
Polk.Two.or.More.Races <- filter(blackhawk_race, 
                         variable == "Two")
```

[^1]: The above quote is excerpted from Rob Pike's [talk](https://www.youtube.com/watch?v=PAAkCSZUG1c) during Gopherfest, November 18, 2015.

### Creating a Map

Tables aren't part of the core Markdown spec, but Hugo supports supports them out-of-the-box.

   Name | Age
--------|------
    Bob | 27
  Alice | 23

#### Inline Markdown within tables

| Italics   | Bold     | Code   |
| --------  | -------- | ------ |
| *italics* | **bold** | `code` |

## Code Blocks

#### Code block with backticks

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Example HTML5 Document</title>
</head>
<body>
  <p>Test</p>
</body>
</html>
```

#### Code block indented with four spaces

    <!doctype html>
    <html lang="en">
    <head>
      <meta charset="utf-8">
      <title>Example HTML5 Document</title>
    </head>
    <body>
      <p>Test</p>
    </body>
    </html>

#### Code block with Hugo's internal highlight shortcode
{{< highlight html >}}
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Example HTML5 Document</title>
</head>
<body>
  <p>Test</p>
</body>
</html>
{{< /highlight >}}

## List Types

#### Ordered List

1. First item
2. Second item
3. Third item

#### Unordered List

* List item
* Another item
* And another item

#### Nested list

* Fruit
  * Apple
  * Orange
  * Banana
* Dairy
  * Milk
  * Cheese
