# Disney Data Exploration using SQL

# Project Overview
This project analyzes Disney+ content data using SQL to uncover key insights about movies and TV shows. It includes distribution by type, popular ratings, top genres, country-wise content count, and contributions by directors and actors. Advanced SQL techniques like CTEs, subqueries, and string functions are used to support business-oriented queries and decision-making.

<img src="dataset-cover.jpg"/>

# Objectives

**1. Set up the Disney+ Content Database**: Create and populate the Disney+ table with attributes such as show ID, type, title, director, cast, country, date added, release year, rating, duration, genres, and description.

**2. Content Analysis Operations**: Perform content-based queries to count and classify movies and TV shows, identify missing data, and retrieve specific content based on filters.

**3. CTEs and Subqueries:** Use Common Table Expressions and subqueries to extract insights like most frequent ratings, top actors, and year-wise averages.

**4. Advanced SQL Queries**: Develop complex SQL queries using ranking functions, string manipulation, aggregation, and filtering to explore content trends, genre distribution, and regional performance.

```sql

--Schemas for Disney+

DROP TABLE IF EXISTS Disney;
CREATE TABLE Disney
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);

-- View table structure (optional)

SELECT * FROM Disney;

--Import Data and check imported data

Select * from Disney

--Business Problems and Solutions

--1. Count the Number of Movies vs TV Shows
Select type, COUNT(*)as total_content
From Disney
Group by type

--Objective: Determine the distribution of content types on Netflix.

--2. Find the Most Common Rating for Movies and TV Shows

--USING CTE
WITH RatingCounts AS (
    SELECT
        [type],
        [rating],
        COUNT(*) AS [rating_count]
    FROM 
        [dbo].[Disney]
    GROUP BY 
        [type], 
        [rating]
),
RankedRatings AS (
    SELECT
        [type],
        [rating],
        [rating_count],
        RANK() OVER (PARTITION BY [type] ORDER BY [rating_count] DESC) AS [rating_rank]
    FROM 
        RatingCounts
)
SELECT
    [type],
    [rating] AS [most_frequent_rating]
FROM 
    RankedRatings
WHERE 
    [rating_rank] = 1;


--USING SUB QUERY
SELECT
    [type],
    [rating]
FROM
(
    SELECT
        [type],
        [rating],
        COUNT(*) AS [rating_count],
        RANK() OVER (PARTITION BY [type] ORDER BY COUNT(*) DESC) AS [ranking]
    FROM 
        [dbo].[Disney]
    GROUP BY 
        [type], 
        [rating]
) AS t1
WHERE
    [ranking] = 1;

--Objective: Identify the most frequently occurring rating for each type of content.

--3. List All Movies Released in a Specific Year (e.g., 2020)
SELECT *
FROM Disney
WHERE
	release_year = 2016

--Objective: Retrieve all movies released in a specific year.

--. Find the Top 5 Countries with the Most Content on Disney+

SELECT
	TOP 5 
	[country], 
	COUNT(*) AS [country_count]
FROM Disney
WHERE
	[country] is not null
GROUP BY 
	[country]
ORDER BY
	[country_count] DESC

--Objective: Identify the top 5 countries with the highest number of content items.

--5. Identify the Longest Movie

SELECT *
	FROM Disney
WHERE
	[type] = 'movie' And 
	[duration] = (
				SELECT
					MAX([duration])
				FROM Disney
				WHERE
					[type] = 'movie' )

--Objective: Find the movie with the longest duration.

--6. Find Content Added in the Last 5 Years

SELECT *
	FROM Disney
WHERE
	CAST([date_added] AS DATE) < DATEADD(DAY, -1825, GETDATE())

--Objective: Retrieve content added to Netflix in the last 5 years.

--7. Find All Movies/TV Shows by Director 'Jack Hannah'

SELECT *
	FROM Disney
WHERE
	[director] = 'Jack Hannah'

--Objective: List all content directed by 'Jack Hannah'

--8. List All TV Shows with More Than 5 Seasons

SELECT * 
FROM Disney
WHERE type = 'TV Show'
And TRY_CAST(LEFT( duration, CHARINDEX(' ', duration + ' ')-1) AS int) > 5

--Objective: Identify TV shows with more than 5 seasons.

--9. Count the Number of Content Items in Each Genre

SELECT 
	LTRIM(RTRIM(value)) as genre,
	COUNT(*) as total_content
FROM Disney
CROSS APPLY string_split(REPLACE(listed_in,',', ','), ',')
WHERE listed_in IS NOT NULL AND listed_in <> ''
GROUP BY LTRIM(RTRIM(value))
ORDER BY total_content DESC;

--Objective: Count the number of content items in each genre

--10.Find each year and the average numbers of content release in United States on disney.

SELECT
	release_year,
	COUNT(show_id) as total_releases,
	ROUND(CAST(COUNT(show_id) AS float)/
	(SELECT COUNT(show_id) FROM Disney WHERE country ='United States')*100,2) AS avg_release
FROM Disney
WHERE country = 'United States'
Group BY country, release_year
ORDER BY avg_release DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY

--Objective: Calculate and rank years by the average number of content releases by India.

--11. List All Movies that are Documentaries

SELECT * 
FROM Disney
WHERE listed_in LIKE '%Documentary'

--Objective: Retrieve all movies classified as documentaries.

--12. Find All Content Without a Director
SELECT * 
FROM Disney
WHERE director IS NULL

--Objective: List content that does not have a director.

--13. Find How Many Movies Actor 'Jim Cummings' Appeared in the Last 20 Years

SELECT * 
FROM Disney
WHERE cast LIKE '%Jim Cummings%'
AND release_year > YEAR(Getdate()) - 20

--Objective: Count the number of movies featuring 'Jim Cummings' in the last 20 years.

--14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in United States

SELECT 
    LTRIM(RTRIM(value)) AS artists,
    COUNT(show_id) AS total_releases
FROM Disney
CROSS APPLY STRING_SPLIT(REPLACE(cast, ', ', ','), ',')
WHERE cast IS NOT NULL 
  AND cast <> '' 
  AND country LIKE '%United States%'
GROUP BY LTRIM(RTRIM(value))
ORDER BY total_releases DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

--Objective: Identify the top 10 actors with the most appearances in US-produced movies.
```
# Reports

- **Database Schema**: Structure of the Disney+ table with fields like show ID, type, title, director, cast, country, release year, rating, duration, and genres.

- **Data Analysis**: Insights into content distribution (Movies vs. TV Shows), most common ratings, genre-wise counts, top countries by content volume, and actor/director contributions.

- **Summary Reports**: Aggregated data on top genres, longest-duration movies, content trends over years, and high-appearance actors in U.S.-produced content.

# Conclusion
The project offers valuable insights into Disney+ content trends using SQL, enabling data-driven decisions on content type, ratings, genres, and regional contributions through efficient querying and analysis.
