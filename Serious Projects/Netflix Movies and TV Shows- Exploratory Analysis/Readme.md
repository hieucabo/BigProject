# 🎥 Netflix Movies and TV Shows- Exploratory Analysis

*** 

## 📅 **Data source**

I get the data from **Kaggle**. The data set present Netflix movies and TV shows released from 1954 to 2022. 

- [Netflix Movies and TV Shows](https://www.kaggle.com/code/surobhipal/netflix-movies-and-tv-shows-exploratory-analysis/data)

***

## 🏁 **Project objectives**

- Clean and analyse the data.
- Have an insight of the data set. 

## 🧐 **What I learned**

This is my second work after the capstone project from the Google Data Analytics course. 
That is I choose an easy project to remind me what to do and how I should start everything? 

- Learn to use ````lubridate```` and ````skimr```` packages.
- Checking the data set to see if it is ready for analyse.
- Analysing the data set. 
- Using ````ggplot```` for visualize.

## 👟 **Let's get started**

I split this into 2 stages, the first stage is to analyse movies and the second one is about TV shows.

### Load the necessary library 

````r
library(dplyr)
library(ggplot2)
library(tidyverse)
library(xlsx)
library(lubridate)
library(skimr)
````

### Import the dataset

````r
movies <- read.csv("~/RStudio File/netflix-tv-shows-and-movies/Best Movies Netflix.csv")
shows <- read.csv("~/RStudio File/netflix-tv-shows-and-movies/Best Shows Netflix.csv")
````

### Analyse Movies 

- **Preparing Data**

First, let's have a look of our data using ````glimpse()````.

````r
glimpse(movies)
````

````r
Rows: 387
Columns: 8
$ index<int> 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22,…
$ TITLE<chr> "David Attenborough: A Life on Our Planet", "Inception", "Forrest Gump", "Anbe Si…
$ RELEASE_YEAR<int> 2020, 2010, 1994, 2003, 2021, 1998, 2012, 2016, 2016, 2010, 2019, 2009, 2004, 201…
$ SCORE<dbl> 9.0, 8.8, 8.8, 8.7, 8.7, 8.6, 8.4, 8.4, 8.4, 8.4, 8.4, 8.4, 8.4, 8.4, 8.3, 8.3, 8…
$ NUMBER_OF_VOTES<int> 31180, 2268288, 1994599, 20595, 44074, 1346020, 1472668, 180247, 14356, 11973, 25…
$ DURATION<int> 83, 148, 142, 160, 87, 169, 165, 161, 60, 84, 65, 170, 143, 176, 98, 229, 113, 16…
$ MAIN_GENRE<chr> "documentary", "scifi", "drama", "comedy", "comedy", "drama", "western", "action"…
$ MAIN_PRODUCTION<chr> "GB", "GB", "US", "IN", "US", "US", "US", "IN", "US", "US", "US", "IN", "IN", "IN…
````

Are there any observations that have missing values??? 

````r
colSums(is.na(movies))
````

````r
          index           TITLE    RELEASE_YEAR           SCORE NUMBER_OF_VOTES        DURATION
              0               0               0               0               0               0
     MAIN_GENRE MAIN_PRODUCTION
              0               0
````

Wow, the data is already too good too use. 

````r
n_distinct(movies$MAIN_GENRE)
unique(movies$MAIN_GENRE)
n_distinct(movies$MAIN_PRODUCTION)
unique(movies$MAIN_PRODUCTION)
````

````r
> n_distinct(movies$MAIN_GENRE)
[1] 15
> unique(movies$MAIN_GENRE)
 [1] "documentary" "scifi"       "drama"       "comedy"      "western"     "action"      "crime"
 [8] "thriller"    "war"         "fantasy"     "romance"     "horror"      "musical"     "animation"
[15] "sports"
> n_distinct(movies$MAIN_PRODUCTION)
[1] 35
> unique(movies$MAIN_PRODUCTION)
 [1] "GB" "US" "IN" "UA" "CD" "TR" "ES" "AU" "JP" "ZA" "HK" "DE" "KR" "CA" "BE" "NO" "NZ" "MX" "FR" "MW"
[21] "TH" "AR" "PS" "HU" "IT" "CN" "PL" "KH" "IE" "BR" "XX" "LT" "NL" "DK" "ID"
````

There are 15 genres and 35 production countries. Besides, let's rename the column for easier analyse.

````r
movies <- movies %>% 
  rename("title" = "TITLE",
         "release_year" = "RELEASE_YEAR",
         "imdb_score" = "SCORE" ,
         "imdb_votes" = "NUMBER_OF_VOTES",
         "duration" = "DURATION" ,
         "genre" = "MAIN_GENRE",
         "country" = "MAIN_PRODUCTION" )
````

- **Analysing and Visualising**

````r
summary(movies)
````

````r
     index          title            release_year    imdb_score      imdb_votes         duration
 Min.   :  0.0   Length:387         Min.   :1954   Min.   :6.900   Min.   :  10139   Min.   : 28.0
 1st Qu.: 96.5   Class :character   1st Qu.:2008   1st Qu.:7.100   1st Qu.:  20513   1st Qu.:103.5
 Median :193.0   Mode  :character   Median :2014   Median :7.400   Median :  45200   Median :122.0
 Mean   :193.0                      Mean   :2011   Mean   :7.509   Mean   : 136521   Mean   :123.4
 3rd Qu.:289.5                      3rd Qu.:2018   3rd Qu.:7.800   3rd Qu.: 153486   3rd Qu.:139.0
 Max.   :386.0                      Max.   :2022   Max.   :9.000   Max.   :2268288   Max.   :229.0
    genre             country
 Length:387         Length:387
 Class :character   Class :character
 Mode  :character   Mode  :character
````

- The earliest release year available in our dataset is 1954, while the most recent one is 2022. The year 2014 had the highest number of movie releases.
- The IMDb scores for the movies in our dataset range from 6.9 to 9, with an average score of 7.5.
- The movie with the fewest number of IMDb votes has 10,139 votes, while the one with the most votes has 2,268,288 votes.
- The duration of movies in our dataset ranges from 28 to 229 minutes. The most common duration is 122 minutes.

````r
movies_genre <- movies %>% 
  group_by(genre) %>% 
  summarise(movies = n()) %>% 
  arrange(desc(movies)

ggplot(movies_genre,aes(x=movies,y=reorder(genre,movies),fill=genre)) +
  geom_col() +
  labs(
    title = "Number of movies by Genre",
    x = "Number of movies",
    y = "Genre"
  )
  ````
  
<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/220166700-0a6b9e06-1b1c-46a5-8435-56136e6bee51.png">
</p>

- Drama is by far the most common genre among the movies in our dataset.
- On the other hand, the genres of War, Animation, and Sports have the fewest number of movies.

````r
movies_country <- movies %>% 
  group_by(country) %>% 
  summarise(movies = n()) %>% 
  arrange(desc(movies)) 


ggplot(movies_country,aes(x=reorder(country,desc(movies)),y=movies,fill=country)) +
  geom_col() +
  labs(
    title = "Number of movies by Country",
    x = "Country",
    y = "Number of movies"
  ) + 
  theme(legend.position = "none")
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/220167150-a54d5957-515c-4ba8-8d17-5c75b2c4b0ab.png">
</p>

- Most of the movies available on Netflix are from the United States (US), India (IN), and Great Britain (GB).

````r
movies_country_genre <- movies %>% 
  group_by(country,genre) %>% 
  summarise(movies = n()) %>% 
  arrange(desc(movies)) %>% 
  top_n(10)
````

````r
country genre       movies
   <chr>   <chr>        <int>
 1 US      drama           56
 2 IN      drama           48
 3 US      comedy          22
 4 IN      thriller        21
 5 US      thriller        21
 6 IN      comedy          20
 7 GB      drama           14
 8 US      documentary     14
 9 IN      romance         12
10 IN      crime            9
````

- It's noticeable that the top 3 countries with the most movies are among the most appearing on this list.
- Surprisingly, the genres of Documentary, Romance, and Crime, which are not among the top 3 genres, also appear in this list.

````r
movies_genre_imdb <- movies %>% 
  group_by(genre) %>% 
  summarise(median_score = median(imdb_score),
            median_votes = median(imdb_votes)) %>% 
  arrange(desc(median_score))
           
movies_genre_imdb
````

````r
genre       median_score median_votes
   <chr>              <dbl>        <dbl>
 1 scifi               8          288960
 2 action              7.8         52338
 3 documentary         7.8         22235
 4 war                 7.7         50150
 5 comedy              7.6         25640
 6 western             7.6        224900
 7 fantasy             7.5         71182
 8 crime               7.4         37528
 9 drama               7.4         54128
10 romance             7.4         23541
11 sports              7.4         21558
12 thriller            7.4         52657
13 horror              7.25       219210
14 animation           7.1        100787
15 musical             7.05        15568
````

- Although the sci-fi genre has a relatively low number of movies in our dataset, it has the highest median score and median vote among all genres.
- The horror and animation genres have a large number of votes but generally do not receive high scores. On the other hand, the documentary and action genres have a high average score (7.8) but a relatively low number of votes.
- The musical genre has both the lowest median score and the fewest median votes among all genres.

````r
movies %>%
  select(title,imdb_score, release_year,country, genre) %>% 
  arrange(desc(imdb_score)) %>% 
  slice(1:5)
````

````r
																	   title imdb_score release_year country       genre
1 David Attenborough: A Life on Our Planet        9.0         2020      GB documentary
2                                Inception        8.8         2010      GB       scifi
3                             Forrest Gump        8.8         1994      US       drama
4                               Anbe Sivam        8.7         2003      IN      comedy
5                       Bo Burnham: Inside        8.7         2021      US      comedy
````

- Out of the top 5 movies ranked by IMDb score, two are from Great Britain, two are from the United States, and the 
last one is from India. Interestingly, the top 2 movies are not from the popular genres.
 
Because there is only 1 drama movie on the list while the drama genre has the most movies, I want to take a deeper look at this genre.

````r
movies %>% 
  select(title,imdb_score, release_year,country, genre) %>% 
  filter(genre == "drama") %>% 
  arrange(desc(imdb_score)) %>% 
  slice(1:5)
````

````r
                        title imdb_score release_year country genre
1                Forrest Gump        8.8         1994      US drama
2         Saving Private Ryan        8.6         1998      US drama
3 Once Upon a Time in America        8.3         1984      US drama
4         Like Stars on Earth        8.3         2007      IN drama
5           Full Metal Jacket        8.3         1987      GB drama
````

It seems that most of the movies in the top 5 are quite old, with 4 out of 5 having been released before 2000. How about movies after 2000?

````r
movies %>% 
  select(title,imdb_score, release_year,country, genre) %>% 
  filter(genre == "drama", release_year > 2000) %>% 
  arrange(desc(imdb_score)) %>% 
  slice(1:5)
````

````r
                  title imdb_score release_year country genre
1   Like Stars on Earth        8.3         2007      IN drama
2               Warrior        8.2         2011      US drama
3                 Queen        8.2         2014      IN drama
4      Paan Singh Tomar        8.2         2012      IN drama
5 Miracle in Cell No. 7        8.2         2019      TR drama
````

That's an interesting observation! It seems that since 2000, Indian drama movies have been particularly well-received, with three of them appearing in the top 5 drama movies based on IMDb scores.

### Analyse TV Shows 

- **Preparing Data**

````r
glimpse(shows)
````

````r
Rows: 246
Columns: 9
$ index             <int> 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 2…
$ TITLE             <chr> "Breaking Bad", "Avatar: The Last Airbender", "Our Planet", "Kota Factory", "Th…
$ RELEASE_YEAR      <int> 2008, 2005, 2019, 2019, 2020, 2021, 2013, 2011, 2006, 1989, 1998, 2022, 2019, 1…
$ SCORE             <dbl> 9.5, 9.3, 9.3, 9.3, 9.1, 9.1, 9.0, 9.0, 9.0, 8.9, 8.9, 8.9, 8.9, 8.8, 8.8, 8.8,…
$ NUMBER_OF_VOTES   <int> 1727694, 297336, 41386, 66985, 108321, 175412, 325381, 87857, 302147, 302700, 1…
$ DURATION          <int> 48, 24, 50, 42, 50, 41, 24, 23, 24, 24, 25, 28, 74, 30, 26, 21, 49, 52, 23, 58,…
$ NUMBER_OF_SEASONS <int> 5, 3, 1, 2, 1, 1, 4, 3, 1, 9, 1, 1, 1, 4, 6, 3, 6, 3, 21, 6, 3, 3, 6, 3, 5, 2, …
$ MAIN_GENRE        <chr> "drama", "scifi", "documentary", "drama", "documentary", "action", "scifi", "dr…
$ MAIN_PRODUCTION   <chr> "US", "US", "GB", "IN", "US", "US", "JP", "JP", "JP", "US", "JP", "GB", "US", "…
````

````r
colSums(is.na(shows))
````

````r
index             TITLE      RELEASE_YEAR             SCORE   NUMBER_OF_VOTES 
		0                 0                 0                 0                 0 
DURATION NUMBER_OF_SEASONS        MAIN_GENRE   MAIN_PRODUCTION 
	     0                 0                 0                 0
````

Wow, the data is already too good too use. 

````r
n_distinct(shows$MAIN_GENRE)
unique(shows$MAIN_GENRE)
n_distinct(shows$MAIN_PRODUCTION)
unique(shows$MAIN_PRODUCTION)
````

````r
> n_distinct(shows$MAIN_GENRE)
[1] 12
> unique(shows$MAIN_GENRE)
 [1] "drama"       "scifi"       "documentary" "action"      "comedy"      "western"     "animation"  
 [8] "crime"       "reality"     "war"         "thriller"    "romance"    
> n_distinct(shows$MAIN_PRODUCTION)
[1] 19
> unique(shows$MAIN_PRODUCTION)
 [1] "US" "GB" "IN" "JP" "CA" "DE" "AU" "KR" "DK" "TR" "IL" "ES" "SE" "FR" "BE" "BR" "NO" "IT" "FI"
````

There is a total of 12 genres and 19 production countries. However, there are fewer genres and production countries in shows compared to movies.

````r
shows <- shows %>% 
  rename("title" = "TITLE",
         "release_year" = "RELEASE_YEAR",
         "imdb_score" = "SCORE" ,
         "imdb_votes" = "NUMBER_OF_VOTES",
         "duration" = "DURATION" ,
         "genre" = "MAIN_GENRE",
         "country" = "MAIN_PRODUCTION",
         "season" = "NUMBER_OF_SEASONS")
````

- **Analysing and Visualising**

````r
summary(shows)
````

````r
index           title            release_year    imdb_score      imdb_votes         duration     
 Min.   :  0.00   Length:246         Min.   :1969   Min.   :7.500   Min.   :  10024   Min.   : 16.00  
 1st Qu.: 61.25   Class :character   1st Qu.:2013   1st Qu.:7.700   1st Qu.:  18553   1st Qu.: 27.25  
 Median :122.50   Mode  :character   Median :2016   Median :8.000   Median :  41943   Median : 44.00  
 Mean   :122.50                      Mean   :2015   Mean   :8.093   Mean   : 101967   Mean   : 41.92  
 3rd Qu.:183.75                      3rd Qu.:2019   3rd Qu.:8.400   3rd Qu.: 112812   3rd Qu.: 51.75  
 Max.   :245.00                      Max.   :2022   Max.   :9.500   Max.   :1727694   Max.   :141.00  
     season         genre             country         
 Min.   : 1.00   Length:246         Length:246        
 1st Qu.: 1.00   Class :character   Class :character  
 Median : 3.00   Mode  :character   Mode  :character  
 Mean   : 3.65                                        
 3rd Qu.: 5.00                                        
 Max.   :21.00
````

- The most recent release year is 2022, while the lastest release year is 1969. The majority of shows were released in 2016.
- The lowest IMDb score is 7.5, while the highest IMDb score is 9.5. On average, shows on this platform have an IMDb score of 8.
- The TV shows with the fewest IMDb votes has 10,024 votes, while the one with the most votes has 1,727,694 votes.
- The longest chapter duration is 141 minutes, while the shortest is only 16 minutes. The majority of chapters are 44 minutes long.

````r
shows_genre <- shows %>% 
  group_by(genre) %>% 
  summarise(shows = n()) %>% 
  arrange(desc(shows))

ggplot(shows_genre,aes(x=shows,y=reorder(genre,shows),fill=genre)) +
  geom_col() +
  labs(
    title = "Number of shows by Genre",
    x = "Number of shows",
    y = "Genre"
  )
  ````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/220170885-5d127001-f20c-47e6-a9a4-47e7964f2f83.png">
</p>

- As same as movies, drama is the most prominent genre in TV shows, followed by sci-fi and comedy. Thriller shows are less common.
- Reality, western, and romance shows are the least common genres on the platform.

````r
shows_country <- shows %>% 
  group_by(country) %>% 
  summarise(shows = n()) %>% 
  arrange(desc(shows)) 


ggplot(shows_country,aes(x=reorder(country,desc(shows)),y=shows,fill=country)) +
  geom_col() +
  labs(
    title = "Number of shows by Country",
    x = "Country",
    y = "Number of shows"
  ) + 
  theme(legend.position = "none")
````

<p align="center">
  <img src="https://user-images.githubusercontent.com/115451301/220171070-9de3fde7-995a-41b3-9a5c-30df83becd0d.png">
</p>

- The majority of movies on Netflix are from the US, Great Britain, and Japan. 
- While India has a large number of movies, it does not have as many shows on the platform.

````r
shows_country_genre <- shows %>% 
  group_by(country,genre) %>% 
  summarise(shows = n()) %>% 
  arrange(desc(shows)) %>% 
  top_n(10)
````

````r
country genre  shows
   <chr>   <chr>  <int>
 1 US      drama     43
 2 US      comedy    28
 3 US      scifi     25
 4 US      action    14
 5 JP      scifi     12
 6 US      crime     11
 7 GB      drama     10
 8 GB      comedy     6
 9 JP      action     6
10 CA      drama      5
````

- The United States dominates most of the genres on Netflix, with Japan and Great Britain following behind.

````r
shows_genre_imdb <- shows %>% 
  group_by(genre) %>% 
  summarise(median_score = median(imdb_score),
            median_votes = median(imdb_votes)) %>% 
  arrange(desc(median_score))
````

````r
genre       median_score median_votes
   <chr>              <dbl>        <dbl>
 1 western             8.6        73624.
 2 reality             8.45       14086.
 3 documentary         8.1        36661 
 4 scifi               8.1        62367 
 5 animation           8.05       44484.
 6 comedy              8          36359 
 7 drama               8          40948.
 8 action              7.95       70658 
 9 crime               7.9        29247 
10 war                 7.9        30377 
11 thriller            7.8       103818.
12 romance             7.7        10102
````

- Western shows have the highest median IMDb score among all genres, followed closely by reality shows.
- The drama genre, despite having the highest number of shows, does not tend to receive high IMDb scores in comparison to other genres.

````r
shows %>%
  select(title,imdb_score, release_year,country, genre) %>% 
  arrange(desc(imdb_score)) %>% 
  slice(1:5)
````

````r
											 title imdb_score release_year country       genre
1               Breaking Bad        9.5         2008      US       drama
2 Avatar: The Last Airbender        9.3         2005      US       scifi
3                 Our Planet        9.3         2019      GB documentary
4               Kota Factory        9.3         2019      IN       drama
5             The Last Dance        9.1         2020      US documentary
````

- Breaking Bad achieved the highest IMDb score, which is a well-known TV show.
- The top 5 shows consist of 3 from the United States, with 2 of them belonging to the drama genre.
- Despite the small number of shows, the documentary genre contributed 2 shows to the top 5 list.

### Summary

- The US has the most shows and movies on Netflix, likely due to the popularity of the American film industry and the fact that Netflix is an American company.
- Drama is the most prominent genre for both shows and movies on Netflix.
- "Breaking Bad," a drama series from the US, has the highest IMDB score on Netflix at 9.5 points.
- The 2020 documentary "David Attenborough: A Life on Our Planet" has the highest IMDB score of all Netflix content, at 9 points.
- Drama, Thriller, and Comedy are the most common movie genres on Netflix, while Sci-Fi surpasses Thriller to become one of the top three genres for shows.
