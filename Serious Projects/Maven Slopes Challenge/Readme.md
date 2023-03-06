# ⛷️ Maven Slopes Challenge - Ski Resorts

*** 

## 📅 **Data source**

I get the data from **Maven Analytics**. 

There are 2 data sets: 

- resorts (all the information you need about the resort available).
- snows (snow percentage by latitude, longitude by months).

## 🏁 **Project objectives**

- Preparing data sets to visualize. 

## 🧐 **What I learned**

- Use ````mice```` package for fill "unknown" values.
- Using many new functions that I never used before such as ````expand_grid````, ````seq()```` and ````mapply()````,...

## 👟 **Let's get started**

### Load the necessary library 

````r
library(dplyr) 
library(tidyverse)
library(lubridate)
library(mice)
````

### Import the dataset

````r
resorts <- read.csv("~/Maven Slopes Challenge/resorts.csv")
snows <- read.csv("~/Maven Slopes Challenge/snow.csv")
````

### Prepare the dataset 

````r
lat_long <- resorts %>% 
  select(Latitude,Longitude)
````

Get the latitude and longitude of all the resorts.

````r
Month <- data.frame(Month = 1:12)
````

Create a month data frame, from 1 to 12. 

````r
month_lat_long <- expand_grid(Month, lat_long)

month_lat_long $Snow <- NA
````

Use expand_grid from the tidyr package to combine 2 data frames. By that, I will have the value of each month for each longitude and latitude.

Continue with the snows data frame, the month format is ‘01-12-22’ (dd-mm-yy). Let’s get only the month. 

````r
snows <- snows  %>%  
  mutate(Month = month(Month))
````

Use rbind to append 2 data frames for getting data snows data. 

````r
snows_full <- rbind(snows,month_lat_long)
````

Using mice() from the mice package to get the snow value for each longitude and latitude monthly.

````r
imp <- mice(snows_full,m = 4, maxit = 50, method = "pmm", seed = 123)

completed_snows <- complete(imp)
````

````r
colSums(is.na(completed_snows))
````

````r
    Month  Latitude Longitude      Snow
        0         0         0         0
````

Join 2 dataframe to get the snow for each location.

````r
month_lat_long_snow <- left_join(month_lat_long,completed_snows, 
                                 by=c('Latitude'='Latitude','Longitude'='Longitude','Month'='Month'))

month_lat_long_snow <- month_lat_long_snow %>% 
  select(-Snow.x) %>% 
  rename(Snow = Snow.y) %>% 
  unique()
````

````r
Month Latitude Longitude  Snow
   <dbl>    <dbl>     <dbl> <dbl>
 1     1     60.9      8.38 100  
 2     1     60.5      8.21 100  
 3     1     47.1      9.83  22.4
 4     1     49.1   -118.   100  
 5     1     61.2     10.5  100  
 6     1     60.7      6.41  97.2
 7     1    -39.7    177.    92.5
 8     1    -36.6    -72.1   11.0
 9     1     47.6     12.9   96.8
10     1     47.7     13.1   15.4
````

I separate the season's beginning month and the season's closing month into 2 columns. And change the month name to the month number.

````r
resort_month <- resorts %>% 
  select(Latitude,Longitude,Season) %>% 
  filter(Season != "Unknown") %>% 
  mutate(Season = case_when(
    Season == "Year-round" ~ "January - December",
    Season == "January" ~ "January - January",
    Season == "February" ~ "February - February",
    Season == "March" ~ "March - March",
    Season == "April" ~ "April - April",
    Season == "May" ~ "May - May",
    Season == "June" ~ "June - June",
    Season == "July" ~ "July - July",
    Season == "August" ~ "August - August",
    Season == "September" ~ "September - September",
    Season == "October" ~ "October - October",
    Season == "November" ~ "November - November",
    Season == "December" ~ "December - December",
    .default = as.character(Season)
  )) %>% 
  separate_rows(Season, sep=", ") %>% 
  separate(Season,c('Begin','End')) %>% 
  mutate(Begin = case_when(
    Begin == "January" ~ "1", 
    Begin == "February" ~ "2", 
    Begin == "March" ~ "3", 
    Begin == "April" ~ "4",
    Begin == "May" ~ "5", 
    Begin == "June" ~ "6",
    Begin == "July" ~ "7", 
    Begin == "August" ~ "8",
    Begin == "September" ~ "9",
    Begin == "October" ~ "10",
    Begin == "November" ~ "11",
    Begin == "December" ~ "12",
    .default = as.character(Begin)
  ),End = case_when(
    End == "January" ~ "1", 
    End == "February" ~ "2", 
    End == "March" ~ "3", 
    End == "April" ~ "4",
    End == "May" ~ "5", 
    End == "June" ~ "6",
    End == "July" ~ "7", 
    End == "August" ~ "8",
    End == "September" ~ "9",
    End == "October" ~ "10",
    End == "November" ~ "11",
    End == "December" ~ "12",
    .default = as.character(End)
  ))

resort_month$Begin <- as.numeric(resort_month$Begin)
resort_month$End <- as.numeric(resort_month$End)
````

Create a month_sequence to create a sequence from the beginning month and the closing month

````r
month_sequence <- function(start_month, end_month) {
  if (start_month <= end_month) {
    return(seq(start_month, end_month))
  } else {
    return(c(seq(start_month, 12), seq(1, end_month)))
  }
}
````

Get the month of each location where there’s a ski season.

````r
resort_month_list <- resort_month %>%
  mutate(Month = mapply(month_sequence,Begin,End)) %>% 
  unnest(Month) %>% 
  unique()
````

````r
resort_month_snow <- left_join(resort_month_list,month_lat_long_snow,
                           by=c('Latitude'='Latitude','Longitude'='Longitude','Month'='Month'))

resort_month_snow <- resort_month_snow %>% 
  select(-Begin,-End)

resort_month_snow$Snow <- as.numeric(resort_month_snow$Snow)
````

````r
summary(resort_month_snow)
````

````r
    Latitude        Longitude             Month             Snow
 Min.   :-45.05   Min.   :-149.7407   Min.   : 1.000   Min.   :  0.39
 1st Qu.: 43.68   1st Qu.:  -0.2422   1st Qu.: 2.000   1st Qu.: 11.81
 Median : 46.36   Median :   8.0609   Median : 3.000   Median : 77.56
 Mean   : 43.60   Mean   :  -9.3840   Mean   : 5.102   Mean   : 59.34
 3rd Qu.: 47.29   3rd Qu.:  12.1637   3rd Qu.:10.000   3rd Qu.:100.00
 Max.   : 67.78   Max.   : 176.8767   Max.   :12.000   Max.   :100.00
````

It’s pretty weird that a place with only 0.39% snow has a skiing season, but the median is reasonable. So I will take it.

````r
lat_long_unknown <- resorts %>%
  filter(Season == "Unknown") %>% 
  select(Latitude,Longitude)

lat_long_unknown_month <- expand_grid(Month, lat_long_unknown)

resort_month_snow_unknown <- left_join(lat_long_unknown_month,month_lat_long_snow,
                               by=c('Latitude'='Latitude','Longitude'='Longitude','Month'='Month')) 

resort_month_snow_unknown_select <- resort_month_snow_unknown %>% 
  filter(Snow >= median(resort_month_snow$Snow))
````

First, select the place where the season is unknown, then add each month for each latitude and longitude. 
Then left join to find the snow of each month for each location. 
After that, we can filter where the snow is greater than the median snow of other locations.

````r
Month Latitude Longitude  Snow
   <dbl>    <dbl>     <dbl> <dbl>
 1     1     45.8      6.97 100  
 2     1     44.2      7.78 100  
 3     1     47.6   -121.   100  
 4     1     62.5      9.62  89.8
 5     1     35.2   -109.   100  
 6     1     40.0    141.    96.8
 7     1     47.1      8.49 100  
 8     1     46.8      6.35  97.6
 9     1     61.3     12.9  100  
10     1     42.4      2.15 100
````

Now let's bind resort_month_snow_unknown_select and resort_month_snow and export it as a CSV file.

````r
full_resort_month_snow <- rbind(resort_month_snow_unknown_select,resort_month_snow)

write.csv(full_resort_month_snow, file = 'Lat_Long_SeasonMonth.csv', row.names = F)
````

