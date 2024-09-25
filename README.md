# Spotify-Advanced-SQL-and-Query-Optimization
![Alt text](Spotify-Logo.png)
This project demonstrates advanced SQL queries for Spotify data analysis, including EDA, window functions, and aggregations. Queries address business problems such as track popularity, album metrics, and views vs. streams. Optimized for PostgreSQL/pgAdmin4, it includes query tuning techniques to enhance performance and efficiency.

## Overview
This project focuses on analyzing a Spotify dataset collected from [Kaggle](https://www.kaggle.com/), leveraging **SQL** to clean, transform, and explore the data. Additionally, **Excel** was used for initial data cleaning to remove inconsistencies before further SQL processing. After cleaning, the dataset was explored through various **SQL queries** to gain deeper insights into the music tracks, artists, albums, and streaming behavior. The queries are divided into multiple sections, starting from basic retrieval to more advanced queries using window functions and aggregate operations. The project was executed using **PostgreSQL** (or **pgAdmin4**) and optimized for performance where necessary.

---

## Table Creation

The table used for this project was created using the following SQL script:

```sql
CREATE TABLE Spotify
(
   Artist VARCHAR(255),
   Track VARCHAR(255),
   Album VARCHAR(255),
   Album_type VARCHAR(50),
   Danceability FLOAT,
   Energy FLOAT,
   Loudness FLOAT,
   Speechiness FLOAT,
   Acousticness FLOAT,
   Instrumentalness FLOAT,
   Liveness FLOAT,
   Valence FLOAT,
   Tempo FLOAT,
   Duration_Min FLOAT,
   Title VARCHAR(255),
   Channel VARCHAR(255),
   Views_count FLOAT,
   Likes_count BIGINT,
   Comment_count BIGINT,
   Licensed BOOLEAN,
   Official_video BOOLEAN,
   Stream BIGINT,
   Energy_Liveness FLOAT,
   Most_Played_on VARCHAR(255)
);
```

---

## Steps in the Project

### 1. **Data Cleaning and Transformation**
Before performing any analysis, the dataset was first cleaned and transformed to ensure accurate and consistent results. The key steps in data cleaning included:
- **Excel for Initial Cleaning**: Removed rows with missing or inconsistent values and formatted columns for easy import into PostgreSQL.
- **SQL Cleaning**: Removed rows with 0 duration and inconsistent album names within the database.

   ```sql
   -- Delete songs with 0 duration
   DELETE FROM Spotify
   WHERE Duration_Min = 0;
   
   -- Delete rows with inconsistent album names
   DELETE FROM Spotify
   WHERE Album = '#NAME?';
   ```

### 2. **Exploratory Data Analysis (EDA)**
Once the data was cleaned, an **EDA** was performed to get an understanding of the dataset. This included counting rows, identifying distinct artists and albums, and calculating the maximum and minimum song durations. Some of the key queries include:

- **Number of Rows**: 
   ```sql
   SELECT COUNT(*)
   FROM Spotify;
   ```
  
- **Distinct Artists**: 
   ```sql
   SELECT COUNT(DISTINCT Artist)
   FROM Spotify;
   ```
  
- **Maximum and Minimum Duration**:
   ```sql
   SELECT Album, Album_type, Artist, Duration_Min
   FROM Spotify
   WHERE Duration_Min = (SELECT MAX(Duration_Min) FROM Spotify);
   ```

---

## 3. **Solving Business Problems**

### Easy Level Queries
1. **Retrieve tracks with more than 1 billion streams**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Stream > 1000000000;
   ```

2. **List all albums along with their respective artists**:
   ```sql
   SELECT DISTINCT Album, Artist
   FROM Spotify;
   ```

3. **Get the total number of comments for tracks where licensed = TRUE**:
   ```sql
   SELECT SUM(Comment_count) AS Total_No_of_Comments
   FROM Spotify
   WHERE Licensed = TRUE;
   ```

4. **Find all tracks that belong to the album type 'single'**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Album_type = 'single';
   ```

5. **Count the total number of tracks by each artist**:
   ```sql
   SELECT Artist,
          COUNT(*) AS Total_No_of_Tracks
   FROM Spotify
   GROUP BY Artist;
   ```

---

### Medium Level Queries
1. **Calculate the average danceability of tracks in each album**:
   ```sql
   SELECT Album,
          AVG(Danceability) AS Average_Danceability
   FROM Spotify
   GROUP BY Album;
   ```

2. **Find the top 5 tracks with the highest energy values**:
   ```sql
   SELECT DISTINCT Track, Album, Artist, Energy
   FROM Spotify
   ORDER BY Energy DESC
   LIMIT 5;
   ```

3. **List all tracks along with their views and likes where official_video = TRUE**:
   ```sql
   SELECT Track,
          SUM(Views_count) AS Views, 
          SUM(Likes_count) AS Likes
   FROM Spotify
   WHERE official_video = TRUE
   GROUP BY Track
   ORDER BY Likes DESC;
   ```

4. **For each album, calculate the total views of all associated tracks**:
   ```sql
   SELECT Album,
          SUM(Views_count) AS Total_Views
   FROM Spotify
   GROUP BY Album;
   ```

5. **Retrieve the track names that have been streamed on Spotify more than YouTube**:
   ```sql
   SELECT *
   FROM (SELECT Track,
                SUM(CASE WHEN Most_Played_on = 'Spotify' THEN Stream ELSE 0 END) AS Stream_Spotify,
                SUM(CASE WHEN Most_Played_on = 'Youtube' THEN Stream ELSE 0 END) AS Stream_Youtube
         FROM Spotify
         GROUP BY Track) AS t 
   WHERE Stream_Spotify > Stream_Youtube 
     AND Stream_Youtube <> 0;
   ```

---

### Advanced Level Queries
1. **Find the top 3 most-viewed tracks for each artist using window functions**:
   ```sql
   WITH ranking_artist AS (
     SELECT Artist, 
            Track, 
            SUM(Views_count) AS Total_Views,
            DENSE_RANK() OVER (PARTITION BY Artist ORDER BY SUM(Views_count) DESC) AS rank
     FROM Spotify
     GROUP BY Artist, Track
   )
   SELECT *
   FROM ranking_artist
   WHERE rank <= 3;
   ```

2. **Find tracks where the liveness score is above the average**:
   ```sql
   SELECT *
   FROM Spotify
   WHERE Liveness > (SELECT AVG(Liveness) FROM Spotify);
   ```

3. **Use a WITH clause to calculate the difference between the highest and lowest energy values for tracks in each album**:
   ```sql
   WITH EnergyLevel AS (
     SELECT Album,
            MAX(Energy) AS Max_Energy,
            MIN(Energy) AS Min_Energy
     FROM Spotify
     GROUP BY Album
   )
   SELECT Album, 
          (Max_Energy - Min_Energy) AS Difference
   FROM EnergyLevel
   ORDER BY Difference DESC;
   ```

4. **Find tracks where the energy-to-liveness ratio is greater than 1.2**:
   ```sql
   SELECT Track,
          (Energy / Liveness) AS Energy_Liveness_Ratio
   FROM Spotify
   WHERE Energy / Liveness > 1.2
   ORDER BY Energy_Liveness_Ratio DESC;
   ```

5. **Calculate the cumulative sum of likes for tracks ordered by the number of views using window functions**:
   ```sql
   SELECT Artist, 
          Track, 
          Views_count, 
          Likes_count,
          SUM(Likes_count) OVER (ORDER BY Views_count DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS Cumulative_Likes
   FROM Spotify
   ORDER BY Views_count DESC;
   ```

---

## Conclusion
This project demonstrates how **Excel** and **SQL** can be used to clean, transform, and analyze large datasets like Spotify's track listings. Through exploratory data analysis and solving various business questions, the project provides insights into music trends, streaming patterns, and song popularity, while utilizing advanced SQL techniques such as window functions and aggregate operations. Optimizations were implemented throughout to ensure the queries performed efficiently.

Feel free to explore the SQL queries and customize them for your own dataset analysis!

---

### Tools and Technologies Used:
- **Excel** for initial data cleaning
- **PostgreSQL / pgAdmin4** for SQL querying
- **SQL** for data analysis
- **Kaggle** for the dataset

### Dataset:
The dataset used for this project is sourced from Kaggle, which includes various attributes of Spotify tracks such as artist, track name, album, streams, and more. You can access the dataset [here](https://www.kaggle.com/datasets/sanjanchaudhari/spotify-dataset).

---
