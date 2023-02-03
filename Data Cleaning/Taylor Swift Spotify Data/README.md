# 🎶 Taylor Swift Spotify Data 

<p align="center">
  <img width="400" height="400" src="https://user-images.githubusercontent.com/115451301/216564110-324c6c66-6e07-4908-87f1-c2ea6ac664e9.jpg">
</p>


I am just a big fan of Taylor Swift (you can call me Swifties 🤣). When searching for data to practice, I bumped into this and decided to have a glance.

***

## 📅 **Data source**

I get the data from **Kaggle**. The dataset presents all Taylor Swift Spotify Data obtained as of 2022-10-27 by Spotify. 

- [Taylor Swift Spotify Data](https://www.kaggle.com/datasets/arthurboari/taylor-swift-spotify-data)

***

## 🏁 **Project objectives**

Take a list of Taylor Swift's songs which are from the standard edition

***

## 🧐 **What I learned**

- I never thought data cleaning would be this complicated since my study before all used cleaned data.

- By the way, it's also challenging and I am passionate when working with it.

***

## 👟 **Let's get started**

In this project, I used **SQL** to clean the data.

I already input the ````.csv```` file into SQL and named the table as ````spotify````.

### Choosing Data

````sql
SELECT
    album_release_date AS date,
    duration_ms AS duration,
    explicit,
    track_name AS song,
    album_name AS album
INTO spotify_select
FROM spotify
````

Because the original has so much data that is not needed. So I only took 5 columns that I think necessary.

### Cleaning Data

````sql
SELECT DISTINCT album
INTO album_table
FROM spotify_select
WHERE album NOT LIKE '%Radio%'
AND album NOT LIKE '%Deluxe%'
AND album NOT LIKE '%Version%'
AND album NOT LIKE '%Edition%'
AND album NOT LIKE '%Karaoke%'
AND album NOT LIKE '%Tour%'
AND album NOT LIKE '%Live%'
ORDER BY album
````

Because the objective is to have the standard edition, so let's remove the additional versions, tour and live. Then put it into a table name album_table to not affect the original data.

|    |     album    |
|:--:|:------------:|
| 1  | 1989         |
| 2  | evermore     |
| 3  | Fearless     |
| 4  | folklore     |
| 5  | Lover        |
| 6  | Midnights    |
| 7  | Red          |
| 8  | reputation   |
| 9  | Speak Now    |
| 10 | Taylor Swift |

This is exactly 10 published albums.

````sql
SELECT DISTINCT album, date 
FROM spotify_select 
WHERE album IN (
    SELECT album 
    FROM album_table
)
ORDER BY album
````

|     album    |   date   |
|:------------:|:--------:|
| 1989         | 01-01-14 |
| 1989         | 27-10-14 |
| evermore     | 10-12-20 |
| evermore     | 11-12-20 |
| Fearless     | 01-01-08 |
| Fearless     | 11-11-08 |
| folklore     | 24-07-20 |
| Lover        | 23-08-19 |
| Midnights    | 21-10-22 |
| Red          | 22-10-12 |
| reputation   | 10-11-17 |
| Speak Now    | 25-10-10 |
| Taylor Swift | 24-10-06 |

After taking only the standard album, there are still some albums with different released dates.  I choose the album with the oldest date because I only want the standard album. 

````sql
SELECT DISTINCT album, MIN(date) AS date
FROM spotify_select 
WHERE album IN (
    SELECT album 
    FROM album_table
)
GROUP BY album
	ORDER BY album
````

|     album    |    date    |
|:------------:|:----------:|
| 1989         | 2014-01-01 |
| evermore     | 2020-12-10 |
| Fearless     | 2008-01-01 |
| folklore     | 2020-07-24 |
| Lover        | 2019-08-23 |
| Midnights    | 2022-10-21 |
| Red          | 2012-10-22 |
| reputation   | 2017-11-10 |
| Speak Now    | 2010-10-25 |
| Taylor Swift | 2006-10-24 |

````sql
WITH cte AS(
    SELECT DISTINCT 
        album, 
        MIN(date) AS date
    FROM spotify_select 
    WHERE album IN (
        SELECT album 
        FROM album_table
    )
    GROUP BY album
)
SELECT DISTINCT
    c.date,
    MAX(duration) AS duration, 
    MAX(CONVERT(int,explicit)) AS explicit,
    song,
    c.album
FROM cte AS c 
LEFT JOIN spotify_select AS s 
ON c.album = s.album AND c.date = s.date 
GROUP BY c.date, song, c.album
ORDER BY album, song
````

Create a temp table containing album and date, and join with spotify_select to list only 
songs with the same date and album name. However, there are songs with the same name, album and 
date but different in duration and explicit. I will choose the song with the longest duration and have explicit for it is the original song. 

|    date    | duration | explicit |                   song                   | album |
|:----------:|:--------:|:--------:|:----------------------------------------:|:-----:|
| 2012-10-22 |  230133  |     0    |                    22                    |  Red  |
| 2012-10-22 | 233003   | 0        | 22 - Karaoke Version                     | Red   |
| 2012-10-22 | 327893   | 0        | All Too Well                             | Red   |
| 2012-10-22 | 330766   | 0        | All Too Well - Karaoke Version           | Red   |
| 2012-10-22 | 237613   | 0        | Begin Again                              | Red   |
| 2012-10-22 | 237600   | 0        | Begin Again - Karaoke Version            | Red   |
| 2012-10-22 | 243933   | 0        | Everything Has Changed                   | Red   |
| 2012-10-22 | 246807   | 0        | Everything Has Changed - Karaoke Version | Red   |

However, there are some duplicates because some song has a different version.

Let's save this into a table name cleaned1.

````sql
WITH cte AS(
    SELECT 
        album, 
        song,
        LAG(song) OVER(PARTITION BY album ORDER BY song) AS pre_song
    FROM cleaned1
),cte2 AS(
    SELECT 
    *,
    CASE WHEN song LIKE '%'+pre_song+'%' THEN 1 
    ELSE 0 END AS same_song
    FROM cte
) 
    SELECT 
        cte. album, cte.song, explicit, date, duration
    FROM cte2 AS cte
    LEFT JOIN cleaned1 AS cle 
    ON cte.album = cle.album AND cte.song = cle.song
    WHERE same_song = 0
````

First I create a temp table, using LAG to create another column that contains the name of the 
song that comes before it. Then if it has the same name as the previous song, count it as 1 (same song). Then select the one with no replicate.

|     album    |              song             | explicit |    date    | duration |
|:------------:|:-----------------------------:|:--------:|:----------:|:--------:|
| Taylor Swift | A Perfectly Good Heart        | 0        | 2006-10-24 | 220146   |
| Taylor Swift | A Place in this World         | 0        | 2006-10-24 | 199200   |
| Taylor Swift | Cold As You                   | 0        | 2006-10-24 | 239013   |
| Taylor Swift | I'm Only Me When I'm With You | 0        | 2006-10-24 | 213053   |
| Taylor Swift | Invisible                     | 0        | 2006-10-24 | 203226   |
| Taylor Swift | Mary's Song (Oh My My My)     | 0        | 2006-10-24 | 213080   |

This looks good, but I still need to check if the number of songs in each album is correct.

````sql
WITH cte AS(
    SELECT 
        album, 
        song,
        LAG(song) OVER(PARTITION BY album ORDER BY song) AS pre_song
    FROM cleaned1
),cte2 AS(
    SELECT 
    *,
    CASE WHEN song LIKE '%'+pre_song+'%' THEN 1 
    ELSE 0 END AS same_song
    FROM cte
) 
    SELECT 
        album, COUNT(song) AS num_song
    FROM cte2 AS cte
    WHERE same_song = 0
    GROUP BY album
````

|     album    | num_song | The real number of songs (only standard edition) | The real number of songs (including extra edition) |
|:------------:|:--------:|:------------------------------------------------:|:--------------------------------------------------:|
| 1989         | 13       | 13                                               | 19 (6 Deluxe)                                      |
| evermore     | 15       | 15                                               | 17 (2 Deluxe)                                      |
| Fearless     | 13       | 13                                               | 26 (6 Platinum, 7 Taylor’s Version)                |
| folklore     | 16       | 16                                               | 17 (1 Deluxe)                                      |
| Lover        | 18       | 18                                               | 18                                                 |
| Midnights    | 13       | 13                                               | 13                                                 |
| Red          | 16       | 16                                               | 22 (6 Deluxe)                                      |
| reputation   | 15       | 15                                               | 15                                                 |
| Speak Now    | 14       | 14                                               | 20 (6 Deluxe)                                      |
| Taylor Swift | 15       | 11                                               | 17 (4 Deluxe, 2 bonus)                             |

Except for Taylor Swift album, the others are true based on the standard edition 
(you can trust me for I’ve been a Swiftie for over 10 years, or go to Wikipedia for extra information). 
In Taylor Swift album, 4 songs are already included even though it is not a standard edition. 

Spotify was founded in 2006 and opened for public registration in 2010. The album Taylor Swift was released in 2006 and 4 out of 6 additional songs were released 
in 2017 and before (2 in 2008). That's why 4 songs were put in the standard album even though it should not to.
