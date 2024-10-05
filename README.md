# Netflix Movies And TV Shows Dataset Analysis Using SQL
![Netflix Logo](https://github.com/DhananjayPimple/netflix_sql_project/blob/main/netflix-logo)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

- **Dataset Link:** [Movies Dataset](https://www.kaggle.com/datasets/shivamb/netflix-shows?resource=download)

## Schema

```sql
CREATE TABLE netflix
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
```

## Business Problems and Solutions

### 1. Calculate the total number of Movies & T.V. shows available on the platform.

```sql
SELECT type, COUNT(*) AS Total
FROM netflix
GROUP BY type  ;
```

### 2. Find the most common rating given to Movies and TV Shows.

```sql
SELECT type, rating 
FROM
(
	SELECT type, rating, COUNT(*),
	DENSE_RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) AS drn
	FROM netflix  
	GROUP BY type, rating 
)AS sub

WHERE sub.drn = 1  ;
```

### 3. Filter out all the data related to Movies released in a specific year. (e.g. 2020)

```sql
SELECT * 
FROM netflix
WHERE type = 'Movie' AND release_year = 2020 ;
```

### 4. Find the top 5 countries according to the amount of content present on Netflix.

```sql
SELECT country, COUNT(show_id) AS Total_Content
FROM netflix
GROUP BY country  ;

/* 
	As we have seen in the output of the above query, we are getting redundant data 
	in the country column. Many countries are present in a group separated by commas, 
	as the same content might be published in all of those countries.

	So the solution to this is: 
	
	First, to convert the data present in the country column from string to array 
	using the "STRING_TO_ARRAY" function.

	Second, break each element present in the array into a separate row using the 
	"UNNEST" function.
*/

SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS new_country
FROM netflix  ;

-- Final answer

SELECT UNNEST(STRING_TO_ARRAY(country, ',')) AS country, 
COUNT(show_id) AS total_content
FROM netflix 
GROUP BY 1  
ORDER BY 2 DESC  
LIMIT 5  ;
```

### 5. Identify the top 5 movies with the longest duration on the platform.

```sql
SELECT title, CAST(SPLIT_PART(duration, ' ', 1) AS INT) AS total_minutes 
FROM netflix
WHERE type = 'Movie' AND CAST(SPLIT_PART(duration, ' ', 1) AS INT) IS NOT NULL
GROUP BY 1, 2
ORDER BY 2 DESC
LIMIT 5  ;
```

### 6. Find all the content added in the last 5 years.

```sql
SELECT *
FROM netflix 
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 YEARS' ;
```

### 7. Find all the Movies/TV Shows directed by a particular director. (e.g. Rajiv Chilaka)

```sql
SELECT *
FROM netflix
WHERE director LIKE '%Rajiv Chilaka%'  ;
```

### 8. List all the TV Shows with more than 5 seasons.

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ',1)::numeric > 5  ;
```

### 9. Count the number of Movies/TV Shows present in each genre.

```sql
SELECT UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre, 
COUNT(show_id) AS Number_Of_Shows_Movies
FROM netflix
GROUP BY 1
ORDER BY 2 DESC  ;
```

### 10. Find each year and the average number of content released by India on Netflix.
###	    Return top five 5 years with the highest number of average content releases.

```sql
SELECT 
EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) AS year, 
COUNT(*) AS yearly_content,
ROUND((COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India')::numeric)*100,2) AS 
percent_content_by_year
FROM netflix
WHERE country = 'India'
GROUP BY 1  ;
```

### 11. List all the movies that are Documentaries.

```sql
SELECT title, listed_in
FROM netflix
WHERE listed_in ILIKE '%Documentaries'  ;
```

### 12. Find all the content on the platform that is without a director.

```sql
SELECT *
FROM netflix
WHERE director IS NULL  ;
```

### 13. Find in how many movies, actor 'Salman Khan' has appeared in the last 10 Years.

```sql
SELECT *
FROM netflix
WHERE casts LIKE '%Salman Khan%' AND 
release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10   ;
```

### 14. Find the top 10 actor/actress who appears most in Movies produced in India.

```sql
SELECT UNNEST(STRING_TO_ARRAY(casts,',')) AS actor_actress,
COUNT(*)
FROM netflix
WHERE country = 'India'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10  ;
```

### 15. Categorize the content based on the presence of the keywords 'kill' and 'violence as Only For Adults and if not as Unrestricted. 
###     Count the number of items for each category.

```sql
WITH CTE AS 
(
	SELECT *, 
	CASE
		WHEN description ILIKE '%kill%' OR description ILIKE '%violence' THEN 'Only For Adults'
		ELSE 'Unrestricted'
		END AS Category
	FROM netflix  
)

SELECT category, COUNT(*) AS total_content
FROM CTE
GROUP BY 1  ;
```














